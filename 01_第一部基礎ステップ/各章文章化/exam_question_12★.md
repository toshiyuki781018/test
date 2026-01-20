# 問題１２：GatewayAPIの作成とHTTPRouteへの移行

## 【問題】

既存のIngressリソースからGateWayAPIへの移行するが、HTTPSアクセスの維持のためHTTPRouteの設定を行ってください。
その上での設定情報は下記になります。

## 【設定情報】

- 既存のIngressリソース `web`
- Namespace `web`
- ホスト名　　 `gateway.web.k8s.local`
- GateWayAPI名 `web-gateway`
- GatewayClass `nginx`（すでにインストールされている想定）
- HTTPRoute名 `wev-route`

## 【■事前準備】

#### ■自己証明書の作成とSecretsの作成
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out tls.crt -keyout tls.key -subj "/CN=gateway.web.k8s.local"
kubectl create secret tls web-tls --cert=tls.crt --key=tls.key
```

#### ■DeploymentとServiceの作成
```bash
kubectl create deployment web-app --image=nginx && kubectl expose deployment web-app --name=web-service --port=80 --target-port=80
```

#### ■IngressコントローラーとIngressリソースのインストール

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.4/deploy/static/provider/baremetal/deploy.yaml
```

```yaml
cat > web-ingress.yaml << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: web
spec:
  ingressClassName: nginx
  rules:
  - host: gateway.web.k8s.local
    http:
      paths:
      - backend:
          service:
            name: web-service
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - gateway.web.k8s.local
    secretName: web-tls
EOF

kubectl apply -f web-ingress.yaml
```

#### ■IPアドレスの確認とホスト名の登録（IngressリソースのIPを確認、その後「/etc/hosts」にホスト名の登録）

```bash
kubectl get ingress 

NAME   CLASS   HOSTS                   ADDRESS         PORTS     AGE
web    nginx   gateway.web.k8s.local   172.31.11.210   80, 443   5m49s
```

```bash
vim /etc/hosts

172.31.11.210 gateway.web.k8s.local
```

#### ■Curlコマンドを使用してHTTPSでアクセス（ポートは443のNodePortでアクセスを実施）

```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller

NAME                       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   NodePort   10.96.64.113   <none>        80:31282/TCP,443:32195/TCP   4m25s

curl -k https://gateway.web.k8s.local:32195

<html><body><h1>Welcome to nginx!</h1></body></html>
※「Welcome to nginx!」が表示されれば成功になります。
```

#### ■GatewayAPIとGatewayClassのインストール

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
```

```yaml
cat > gatewayclass.yaml << EOF
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: "nginx.org/gateway-controller"
EOF

kubectl apply -f gatewayclass.yaml 
```
#### ■参考となるk8sドキュメント

- 回答箇所での記載


## 【▼回答】

#### ▼k8sドキュメントからGatewayAPIのフォーマットをコピーと適用・確認

> [GatewayAPIのフォーマット](https://kubernetes.io/blog/2024/05/09/gateway-api-v1-1/#gateway-client-certificate-verification-https-gateway-api-sigs-k8s-io-geps-gep-91)

```yaml
vim gateway-api.yaml

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: gateway.web.k8s.local
    tls:
      mode: Terminate
      certificateRefs:
      - kind: secret
        name: web-tls

kubectl apply -f gateway-api.yaml
```

```bash
kubectl get gateways 

NAME          CLASS   ADDRESS   PROGRAMMED   AGE
web-gateway   nginx             Unknown      11m
```

#### ▼HTTPRouteのフォーマットコピーと適用・確認

> [HTTPRouteフォーマット](https://kubernetes.io/docs/concepts/services-networking/gateway/#api-kind-httproute)

```yaml
vim http-route.yaml

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
  - name: web-gateway
  hostnames:
  - "gateway.web.k8s.local"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: "/"
    backendRefs:
    - name: web-service
      port: 8080

kubectl apply -f http-route.yaml
```

```bash
kubectl get httproute

NAME        HOSTNAMES                   AGE
web-route   ["gateway.web.k8s.local"]   2m11s
```

#### ▼Ingressリソースの削除

```bash
kubectl delete ingress web
```


## ★解説
```
実際のCKAでは事前に設定されているものばかりですので、GatewayAPIの適用とHTTPRouteの適用、この2つができれば
問題を回答するのはそこまで難しくないです。

```
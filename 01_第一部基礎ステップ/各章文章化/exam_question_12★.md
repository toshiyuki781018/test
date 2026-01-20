# 問題１２：GatewayAPIの作成とHTTPRouteへの移行

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

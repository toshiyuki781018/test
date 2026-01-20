# 問題８：Ingressリソース

#### ■参考となるk8sドキュメント
> [Ingressリソース](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

## 【▼回答】

#### ▼IngressClassの確認（IngressClassを指定する必要があるため）
```bash
kubectl get ingressclass

NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       4m8s
```

#### ▼Ingressリソースの作成と適用
```yaml
cat > ingress-resource.yaml << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /ingress-path
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
EOF
```

```bash
kubectl apply -f ingress-resource.yaml
```

## ★解説
```
作成した後、対象となるCurlコマンドを使用して、問題文に記載されている「200」の値が戻ってくれば問題無し
疎通確認を行うCurlコマンドは問題文に記載があります

Ingressリソースの設定方法とYamlファイルのある箇所を
把握しておけば難しい問題ではないです。


```

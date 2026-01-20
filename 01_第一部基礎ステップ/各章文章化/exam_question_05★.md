# 問題５：DeploymentのPort指定とServiceリソース設定による外部解放

#### ■参考となるk8sドキュメント
> [コンテナのポート設定](https://kubernetes.io/docs/tutorials/services/connect-applications-service/#exposing-pods-to-the-cluster)

## 【▼回答】

#### ▼DeplopymentへContainerPortの追記（Editコマンドを使用して♯の箇所に追加）
```bash
kubectl edit deployment front-end -n spline-reticulator
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-04-15T02:03:17Z"
  generation: 1
  labels:
    app: front-end
  name: front-end
  namespace: spline-reticulator
  resourceVersion: "71319"
  uid: 25b1031b-40d8-4df6-8759-30f73e34bfa8
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: front-end
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: front-end
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        ports:              # 追加箇所
        - containerPort: 80 # 追加箇所
          protocol: TCP     # 追加箇所
```

#### ▼Serviceリソースの作成(コマンド)
```bash
kubectl expose deployment nginx-deploy --name=front-end-service --type=NodePort --port=80 --target-port=80 -n spline-reticulator
```

#### ▼Serviceリソースの作成(yamlファイル)
```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: front-end
  name: front-end-service
  namespace: spline-reticulator
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    app: front-end
  type: NodePort
status:
  loadBalancer: {}
```

## ★解説
```
すでにスケジューリングされているDeploymentにEditコマンドを使用して、Portの追記を行い
Serviceリソースを設定すればよいのでそこまで難しい問題ではないです。

```

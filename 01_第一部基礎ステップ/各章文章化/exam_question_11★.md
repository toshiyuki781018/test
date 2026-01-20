# 問題１１：PriorityClassの作成とPriorityClassNameの適用

#### ■参考となるk8sドキュメント

> [PodPriorityClassの作成](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#example-priorityclass)

> [PodPriorityClassNameの追加](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#pod-priority)


## 【▼回答】

#### ▼PodPriorityClassの作成と確認（「user-high-priority」の値から1引いたPriorityClassを作成）
```bash
kubectl create priorityclass high-priority --value=799999 --description="Priotity class for user defined high-priority user workload"
```

```bash
kubectl get priorityclass 

NAME                      VALUE        GLOBAL-DEFAULT   AGE
high-priority             799999       false            10s
system-cluster-critical   2000000000   false            24h
system-node-critical      2000001000   false            24h
user-high-priority        800000       false            7s
```

#### ▼kubectl editコマンドを使用して対象となるDeploymentにPodPriorityClassの追加を実施（PriorityClassNameの追加）
```yaml
kubectl edit deployment -n priority busybox-logger

apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
    kubectl.kubernetes.io/last-applied-configuration: |
  name: busybox-logger
  namespace: priority
  resourceVersion: "42566"
  uid: 27764b07-a7ee-4184-8a90-193c0e45588b
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: busybox-logger
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: busybox-logger
    spec:
      priorityClassName: high-priority # この箇所を追加
      containers:
      - args:
        - while true; do echo "$(date) - Log message from busybox-logger"; sleep 5;
          done
        command:
        - /bin/sh
        - -c
        image: busybox:stable
        imagePullPolicy: IfNotPresent

```

```
PriorityClassを付与した後で、Errorが発生するのでそのエラーが発生するかを確かめる必要あり
```

## ★解説
```
PriorityClassの作成、PriorityClassNameの指定。この2つを実施できればそこまで難しい問題ではないです。
それに加えて、ベースとなるPriorityClassから値をマイナス１引いた値をせっていできればよし
```


# 問題３：Deploymentのリソース設定

## 【問題】

k8s上で管理しているWordPressはDeploymentで3つのPodを起動していますが、一部のPodが稼働しておりません。すべてのPodへリソース要求の設定を行う必要があります。
下記設定内容や条件を確認の上で、設定を行いなさい

## 【条件】
- 3つあるPod（Deployment）の容量を均等に配分
- コンテナとInitコンテナの両方に同じリクエストを設定
- ResourceRequestsの更新
- 更新後はWordPressは3つのレプリカ数を保持し、すべてのPodが稼働している状態であること

## 【■事前準備】

#### ■WordPressのDeployment作成
```bash
cat > wordpress-deploy.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wordpress
  strategy: {}
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      initContainers:
      - name: init-wordpress
        image: busybox
        command: ["sh","-c","echo Initialization && sleep 5"]
      containers:
      - image: wordpress:latest
        name: wordpress
        ports:
        - containerPort: 80
EOF
```


```bash
kubectl apply -f wordpress-deploy.yaml
```


#### ■参考となるk8sドキュメント

> [コンテナリソース](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1)


## 【▼回答】

#### ▼Deploymentのレプリカ数を0に設定、その後確認(一度Pod自体を排除して、再設定の必要があるため)
```bash
kubectl scale deployment wordpress --replicas=0

kubectl get deployments.apps

NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   0/0     0            0           3m58s
```

#### ▼対象となるNodeのリソースに記載されている「Allocatable」の箇所の「cpu」と「memory」値を確認
```bash
kubectl describe node controlplane

Allocatable:
  cpu:                2
  ephemeral-storage:  6482880911
  hugepages-2Mi:      0
  memory:             3903720Ki
  pods:               110
```

#### ▼割り当てられたリソース（Allocated resources）を確認する
```
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                150m (15%)  0 (0%)
  memory             100Mi (5%)  0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)

CPUが150m Memoryが100Mi
```

#### ▼Podに割り当てるCPUとMemoryの値の出し方
```
CPU：2 となっているので、2000になり、基本として使用している1割の値をマイナスし、現在使用している150mをマイナス
その上で3つのレプリカに割り当てるので3で割ったものが1つ当たりに割り当てるCPUになります。
2000-200=1800
1800-150=1650
1650/3=550
割り当てるCPUは550m

Memoryは 3903720Kiとなっているのでまず、KiからMiに値の変更を行い、基本使用している1割をマイナスし、現在使用しているメモリ100Miをマイナスする
3903720/1024=3812　
3812/10=381
3812-381=3431
3431-100=3331
3331/3=1110

出た値が1024以上であれば、1110/1024=1としてGiの表記にして記載、1024以下であればMi表記で記載する
ここでは1110以上なので、1Giとの記載を行うようにする。

※Linux上で計算するのであれば「expr」コマンドを使用するのが便利です
expr 1146 / 1024
1
※計算するにあたっては行間を開ける必要があるので注意
```

#### ▼割り当てられるCPUとMemoryの値
```
cpu: 350m
Memory: 1Gi
```

#### ▼EditコマンドでResource/requestsの値を設定（注意点はInitContainerもあるので、そこにも割り当てるように注意する）
```bash
kubectl edit deployment wordpress 
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
  creationTimestamp: "2025-04-04T07:51:07Z"
  generation: 2
  name: wordpress
  namespace: default
  resourceVersion: "9240"
  uid: 73166823-7bf9-462e-a4de-7aa51fe6c130
spec:
  progressDeadlineSeconds: 600
  replicas: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: wordpress
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: wordpress
    spec:
      containers:
      - image: wordpress:latest
        imagePullPolicy: Always
        name: wordpress
        ports:
        - containerPort: 80
          protocol: TCP
        resources: 
          requests:
            cpu: 350m     # 修正箇所
            memory: 1Gi  # 修正箇所
      initContainers:
      - command:
        - sh
        - -c
        - echo Initialization && sleep 5
        image: busybox
        imagePullPolicy: Always
        name: init-wordpress
        resources:
          requests:
            cpu: 350m    # 修正箇所
            memory: 1Gi  # 修正箇所
```
#### ▼ scaleコマンドの実行(コマンドを実行後、レプリカ3つが起動すれば成功になります。)
```bash
kubectl scale deployment wordpress --replicaset=3

```


## ★解説と注意点

```
問題を回答する手順は上記の通りになります。
ただ実際の問題で違うのは、割り当てているリソースがCPUはかなり多め、メモリは一般的なので、
そこの計算をきちんとして割り当てる必要があるので注意

kubectl editコマンドにて編集を行う際
Resourcesの中にRequestsとLimitが割り当てられている。
変更箇所はRequestsのみなのでそこを注意するようにする
あと、変更箇所は、通常のコンテナとInitコンテナの2つの箇所のRequestsを変更する必要があるのでそこを注意する

```
```
※注意点
この問題はあくまで疑似的に問題を解くための手法として、例を挙げています。実際の問題は問題文や
出てくる内容にかなり違いがあるので、そこを認識しておくようにしてください。
```
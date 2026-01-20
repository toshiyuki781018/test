# 問題４：sidecarコンテナの追加　

#### ■参考となるk8sドキュメント

> [サイドカーコンテナ](https://kubernetes.io/ja/docs/concepts/cluster-administration/logging/#sidecar-container-with-logging-agent)

## 【▼回答】

#### ▼Yamlファイルの抽出とVimコマンドの実行
```bash
kubectl get deployment synergy-leverager -o yaml > sidecar.yaml

vim sidecar.yaml
```

#### 抽出したサイドカーコンテナYamlへの追記(♯の箇所を追記する)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
  name: synergy-leverager
  namespace: default
  resourceVersion: "38762"
  uid: f2b0946f-ad6a-486d-b618-f01f1094e537
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: synergy-leverager
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: synergy-leverager
    spec:
      volumes:　　　　 #
      - name: varlog #
        emptyDir: {} #
      containers:
      - env:
        - name: LOG_FILENAME
          value: /var/log/synergy-leverager.log
        image: lfcert/monitor:latest
        imagePullPolicy: Always
        name: monitor
        volumeMounts:　　　　　　 #
        - name: varlog　　　　　 #
          mountPath: /var/log  #
      - name: sidecar                                                             #
        image: busybox:stable                                                     #
        command: ["/bin/sh", "-c", "tail -n+1 -f /var/log/synergy-leverager.log"] #
        volumeMounts:                                                             #
        - name: varlog                                                            # 
          mountPath: /var/log                                                     #
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30

```


#### ▼追記したサイドカーコンテナ確認
```bash
kubectl get pod 

NAME                                 READY   STATUS      RESTARTS   AGE
synergy-leverager-84557b5d5b-tskx6   2/2     Running     0          8s
```

#### ▼Logの排出確認
```bash
kubectl logs pod/synergy-leverager-84557b5d5b-tskx6 -c sidecar

2025/04/08 04:02:56 WARN Increased Memory Usage
2025/04/08 04:02:58 INFO Application state is healthy
2025/04/08 04:03:00 WARN Long Running DB Query
2025/04/08 04:03:02 ERROR NullPointerException
2025/04/08 04:03:04 WARN Long Running DB Query
2025/04/08 04:03:06 ERROR NullPointerException
2025/04/08 04:03:08 ERROR Error on executing DB Query
2025/04/08 04:03:10 INFO Application state is healthy
2025/04/08 04:03:12 ERROR NullPointerException
2025/04/08 04:03:14 WARN Increased Memory Usage
2025/04/08 04:03:16 WARN Increased Memory Usage
```

## ★解説
```
サイドカーコンテナを使用することは確かなのだが、InitContainerを使用するとそもそも起動に失敗するので
意味がない。そのため、InitContainerを使用せずにサイドカーコンテナを作成する必要があるのでそこを注意しましょう。

Deploymentのハッシュ値は作成したDeploymentによって変わるのでそこを注意する


```

# 問題７：PersistentVolumeClaimの作成


#### ■参考となるk8sドキュメント

> [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)


## 【▼回答】

#### ▼PersistentVolumeClaimの作成・適用・確認
```yaml
・PVCの作成
cat > pvc-mariadb.yaml << EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb
  namespace: mariadb
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
  storageClassName: local-path
EOF
```

```bash
kubectl apply -f pvc-mariadb.yaml
```

```bash
kubectl get pvc -n mariadb
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mariadb   Pending                                      local-path     <unset>                 21s
※この時点では、STATUSはPendingになっている
```

#### ▼YamlファイルにPVC名の記載とApply（mariadb-deploy.yaml）
```bash
vim mariadb-deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  strategy: {}
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - image: mariadb:latest
        name: mariadb
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mariadb-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-storage
        persistentVolumeClaim:
          claimName:　""          # ここだけ歯抜け状態になっているので、「mariadb」と記載してApplyすればよし
```

```bash
kubectl apply -f mariadb-deploy.yaml
```

#### ▼事前に作成しているPVCがBoundされ、かつPodが動作しているかを確認する
```bash
kubectl get pod,pvc -n mariadb
NAME                          READY   STATUS    RESTARTS   AGE
pod/mariadb-647bbf879-knzdl   1/1     Running   0          14s

NAME      STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mariadb   Bound    mariadb-pv   300Mi      RWO            local-path     <unset>                 22m
※STATUSが「Pending」⇒「Bound」になっていれば成功
```

## ★解説
```
PersistentVolumeClaimとPersistentVolumeの関係をわかっている上で、ドキュメントを確認しながら
作成を実施すれば特に難しい問題ではないですね
```



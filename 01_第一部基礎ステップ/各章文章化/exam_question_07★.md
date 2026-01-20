# 問題７：PersistentVolumeClaimの作成（作成したStorageClassを使用）

## 【問題】

指定されたNamespaceのDeploymentが誤って削除されてしまった。データの永続性を確保しつつ、Deploymentを復元する必要がある。
以下設定内容で復元を実施しなさい。尚、StorageClassは先程作成した「local-path」を使用しなさい

## 【設定情報】

- Namespace `mariadb`
- PersistenVolume `mariadb-pv`
- persistenVolumeClaim `mariadb`
- Accessモード `ReadWriteOnce`
- Storageの容量 `250Mi`
- StorageClass `local-path`
- 適用するYamlファイル `mariadb-deploy.yaml　`

## 【■事前準備】

#### ■Namespaceの作成
```bash
kubectl create namespace mariadb
```

#### ■persistenVolumeの作成・適用・確認
```yaml
cat > maria-pv.yaml << EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv
spec:
  capacity:
    storage: 300Mi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-path
  hostPath:
    path: "/tmp/mariadb-data"
EOF
```

```bash
kubectl apply -f maria-pv.yaml

kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
mariadb-pv   300Mi      RWO            Retain           Available           local-path     <unset>                          7s
```

#### ■最後に適用するDeploymentファイルのDL
```bash
wget -o- https://gitlab.com/t-tashibu/yaml/-/raw/main/mariadb-deploy.yaml
```

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

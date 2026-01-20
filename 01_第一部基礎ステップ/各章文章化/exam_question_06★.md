# 問題６：StorageClassの追加

#### ■参考となるk8sドキュメント

> [StorageClassの設定](https://kubernetes.io/docs/concepts/storage/storage-classes/#storageclass-objects)

## 【▼回答】

#### ▼StorageClassのYamlファイルを作成を適用（Default設定の追加箇所は「Annotation」の箇所）
```yaml
cat > storageclass.yaml << EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
EOF
```

```bash
kubectl apply -f storageclass.yaml
```

#### ▼適用したStorageClassの確認（適用したStrorageclassの横に「default」と記載があればよし）
```bash
kubectl get storageclass

NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  3s
```

## ★解説
```
この問題は作成したYamlファイルに「storageclass.kubernetes.io/is-default-class: "true"」を追記することがわかっていれば
特に難しい問題ではない。ドキュメントの箇所を覚えておいて、それを追記できればよし

```


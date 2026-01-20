# 問題１５：CRDの一覧取得とJsonPathから指定の値を取得する方法

## 【問題】

対象の環境にはCRDがスケジューリングされています。その中でCert-managerと記載されたすべてのCRDの一覧を取得して
custom-crd.yamlへの保存を保存を実施しなさい。
その上でcert-managerからSubjectsフィールドを取得して、cert-manager-subject.yamlへの保存を実施しなさい。

## 【■事前準備】

#### ■対象となるCRDのスケジューリング
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.crds.yaml
```

## 【▼回答】

#### ▼カスタムリソースCRDであるCert-manager一覧の取得と指定のファイルへの保存
```bash
kubectl get crd | grep cert

certificaterequests.cert-manager.io   2025-04-10T08:13:43Z
certificates.cert-manager.io          2025-04-10T08:13:43Z
challenges.acme.cert-manager.io       2025-04-10T08:13:43Z
clusterissuers.cert-manager.io        2025-04-10T08:13:43Z
issuers.cert-manager.io               2025-04-10T08:13:43Z
orders.acme.cert-manager.io           2025-04-10T08:13:43Z
```

```bash
kubectl get crd | grep cert > custom-crd.yaml
```

#### ▼subjectを取得するCert-managerの特定
```bash
kubectl get crds certificates.cert-manager.io

NAME                           CREATED AT
certificates.cert-manager.io   2025-04-10T08:13:43Z
```

#### ▼subjectの取得（コマンドからの取得方法）
```bash
kubectl get crds certificates.cert-manager.io -o jsonpath={.spec.versions[*].schema.openAPIV3Schema.properties.spec.properties.subject}　> cert-manager-subject.yaml

```

#### ▼subjectの取得（Yamlファイルから検索）
```
JsonPathをコマンドから指定する場合には、そのコマンドを覚えておかなければいけないので、そこが厄介になります。
なので、Yamlファイルから検索する場合には以下Yamlファイルの構造から取得する値を覚えておけばよいです。
```

```
jsonpath={.spec.versions[*].schema.openAPIV3Schema.properties.spec.properties.subject}
.spec
.versions[*]
.schema
.openAPIV3Schema
.properties
.spec
.properties
.subject

これを階層別に表すと
spec:                       8
  versions:[*] 　　　　　　　 7　　　#リスト項目があるため、[*]が必要になる
    schema:                 6
      openAPIV3Schema:      5
        properties:         4
          spec:             3
            properties:　　　2
              subject:      1
```

#### ▼subjectの取得（ドキュメントサイトを参考にする方法）

> [CRD参考ドキュメントサイト](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#crd-selectable-fields)
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: shirts.stable.example.com
spec:                       # ★
  group: stable.example.com
  scope: Namespaced
  names:
    plural: shirts
    singular: shirt
    kind: Shirt
  versions:　　　　　　　　　　# ★
  - name: v1
    served: true
    storage: true
    schema:　　　　　　　　　　# ★
      openAPIV3Schema:　　　 # ★
        type: object
        properties:　　　　　 # ★
          spec:　　　　　　　　# ★
            type: object
            properties:　　　# ★
              color:
                type: string
              size:
                type: string
```

```
ドキュメントサイトのYamlファイルフォーマットを参考にJsonPathで値を取得していく方法もあります。
```


## ★解説
```
CRDのYamlファイル構造を理解し、JsonPathから値の取得を行うことができればそこまで難しい問題ではないですね。
ただ、その意味が分からないと問題すら解くのが難しくなってしまうのでそこを注意してもらえればよいです。
```
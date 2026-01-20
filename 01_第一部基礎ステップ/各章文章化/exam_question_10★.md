# 問題１０：Helmによるインストールとテンプレート作成

## 【▼回答】

#### ▼ArgoCDのHelmリポジトリURLの確認とレポジトリ作成とアップデート
```
CKAの問題文にはArgoCDのHelmリポジトリのリンクが記されていません。問題文の上記に記載されているArgoCDのリンクがあり、そのリンク先のアクセスURLがHelmのリポジトリ
に使用できるので、まずはアクセスを行いURLを取得することから始めます。
```

> [ArgoCD-Helmリポジトリ](https://argoproj.github.io/argo-helm/)

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

#### ▼リポジトリに登録されているチャートの確認（使用するチャートは上から2番目の「argo/argo-cd」）
```bash
helm search repo argo

NAME                            CHART VERSION   APP VERSION     DESCRIPTION
argo/argo                       1.0.0           v2.12.5         A Helm chart for Argo Workflows
★ argo/argo-cd                 7.8.21          v2.14.8         A Helm chart for Argo CD, a declarative, GitOps...
argo/argo-ci                    1.0.0           v1.0.0-alpha2   A Helm chart for Argo-CI
argo/argo-events                2.4.14          v1.9.5          A Helm chart for Argo Events, the event-driven ...
argo/argo-lite                  0.1.0                           Lighweight workflow engine for Kubernetes
argo/argo-rollouts              2.39.5          v1.8.2          A Helm chart for Argo Rollouts
argo/argo-workflows             0.45.12         v3.6.5          A Helm chart for Argo Workflows
argo/argocd-applicationset      1.12.1          v0.4.1          A Helm chart for installing ArgoCD ApplicationSet
argo/argocd-apps                2.0.2                           A Helm chart for managing additional Argo CD Ap...
argo/argocd-image-updater       0.12.1          v0.16.0         A Helm chart for Argo CD Image Updater, a tool ...
argo/argocd-notifications       1.8.1           v1.2.1          A Helm chart for ArgoCD notifications, an add-o...

```

#### ▼Helmテンプレートの作成

```bash
helm template argo argo/argo-cd --version=7.7.3 --namespace=argocd --set=crds.install=false > ~/argo-helm.yaml
```

```
ここでは、「helm template」コマンドの使用
名称である「argo」、チャートである「argo/argo-cd」の指定
バージョンである「7.7.3」の指定
作成するテンプレートからCRDの項目を除外する「--set=crds.install=false」のオプション指定
これらを実施して、指定のディレクトリにYamlファイルをエクスポートする
```

#### ▼CRDの除外確認（レスポンスが帰ってこなければOK）

```bash
cat ~/argo-helm.yaml | grep CustomResourceDefinition
```


#### ▼Helm installコマンドにてArgoCDのインストールの実施と確認
```bash
helm install argo argo/argo-cd --version=7.7.3 --namespace=argocd --set=crds.install=false
```

```bash
helm list --namespace=argocd

NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
argo    argocd          1               2025-04-03 07:30:31.695538372 +0000 UTC deployed        argo-cd-7.7.3   v2.13.0
```

```
最終的にHelmコマンドにてインストールしたので、Helm listコマンドでインストールした履歴が出てくれば成功になります。
この問題は対象のテンプレートYamlファイルの作成とHelmにてインストールするので、Helm listコマンドでインストールしたArgoCDが指定のバージョンで
表示されれば成功になります。
```


## ★解説
Helmを使用してArgoCDのインストールを行う問題になるが、テンプレートの作成が必要であることに加え、そのテンプレートからCRDを除外したものを
作成する必要があることがこの問題の焦点になります。

Helmのオプションである「--set=crds.install=false」であるCRDを除外するコマンドとバージョンを指定することを覚えておけばそこまで
難しい問題ではないです。

あと注意点としては、作成したテンプレートYamlファイルからもkubectlコマンドを使用すればArgoCDのインストールは実施できるが
Helmコマンドからはインストールした履歴は残らないので、Helm installコマンドを使用してインストールを実施する必要がある。


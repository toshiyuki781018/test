# 問題１３：3つのNetworkPolicyから条件に合うものを適用する問題

## 【問題】

２つのDeploymentがNameSpace「frontend」「banckend」にそれぞれインストールされている。
NameSpace、「frontend」「banckend」の間に相互通信を行うためのNetWorkPoliyの適用を行いなさい。
条件として、「frontend」「banckend」の相互間通信を行うために必要なPolicyは最も制限の少ないポリシーを利用することになります。

指定のディレクトリに3つのNetworkPolicyを記載したYamlファイルがありますので、条件に見合うYamlファイルの適用を実施しなさい。


## 【設定情報】特になし

## 【■事前準備】

#### ■参考となるk8sドキュメント
> [NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

## 【▼回答】

#### ▼3つのNetworkPolicyの特徴

- １：すべてのPod Selectorであり、タイプはIngressですべてのPod SelectorがあるYamlファイル
- ２：Pod SelectorとNameSpace SelectorがあるYamlファイル
- ３：Pod Selector、NameSpace Selector、Pod CIDRがあるYamlファイル

#### ▼解答
```
1は制限がなさすぎてＮＧ
3は制限しすぎていてＮＧ

よって適度な制限がある2のYamlファイルをApllyするのが正解になるので、２のNetworkPolicyのYamlファイルを適用すればOK
```

```
■問題 3つのNetWorkPoliyのYamlファイルから選択をする方法　

そのPolicyを指定された3つの中からあるNetWorkPoliyを反映させてください。
対象となるNetWorkPoliyのYamlファイルはすでも用意されています。

```

## ★解説
```
過度の制限は必要なし、でも最低限のポリシーは必要と問題文に記載があるので適用されるYamlファイルは2つ目のYamlファイルになるので
その適用を実施すればOK

時期によって問題の概要が変わる可能性があるので、問題を解く上でYamlファイルの確認はきちんと行うべきです。
```


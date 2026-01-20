# 問題１３：NetWorkPolicy適用

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



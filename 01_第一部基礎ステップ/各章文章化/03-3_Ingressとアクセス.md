# 3-3 ハンズオン：Ingress とアクセス確認
― 外から入る責務はどこにあるのか ―

## ■ハンズオンの目的

このハンズオンでは、アプリは通信方法を知らなくてよい
という前提を、Ingress を通して体感します。

ここで確認したいのは、Service と Ingress の役割の違い
「外から来る」という責務がどこに置かれているかです。


## 0. 前提確認

これまでのハンズオンで、
- Deployment（nginx）
- Service（ClusterIP）
が存在している前提で進めます。

#### 確認します。
```bash
kubectl get deployment
kubectl get service
```
sample-deployment と sample-service があれば OK です。


## 1. Service の役割を確認する
― 内部向けの接続点 ―

#### まず、Service 経由でアクセスできることを再確認します。
```bash
kubectl run curl --rm -it --image=curlimages/curl --restart=Never -- \
  curl http://sample-service
```

#### ▼観察ポイント
- アクセス先は Pod ではない
- Service 名だけで通信できている
- Pod の数や入れ替わりは意識していない

ここで一度整理します。

Service は「クラスタ内部の接続点」外部公開の仕組みではありません。


## 2. Ingress Controller の存在を確認する

Ingress は、Ingress リソース単体では動きません。

必ず、Ingress Controllerという実体が必要です。

#### KillerCoda 環境では、あらかじめ Ingress Controller が用意されています。確認します。
```bash
kubectl get pod -A | grep ingress
```

#### 🔍 観察ポイント
- Ingress は「設定」
- Controller が「実行役」
この分離が重要です。

## 3. Ingress を作成する

― 外部からの入口を定義する ―

#### Ingress リソースを定義します。
```bash
cat <<EOF > ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-ingress
spec:
  rules:
  - host: sample.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sample-service
            port:
              number: 80
EOF
```

#### 適用します。
```bash
kubectl apply -f ingress.yaml
```

#### Ingress を確認します。
```bash
kubectl get ingress
```

4. Ingress 経由でアクセスする

#### KillerCoda では、以下のように Host ヘッダを指定してアクセスします。
```bash
kubectl run curl --rm -it --image=curlimages/curl --restart=Never -- \
  curl -H "Host: sample.local" http://localhost
```

nginx のレスポンスが返ってくれば成功です。

## 5. Pod を削除してみる

― アクセスはどうなるか ―

#### Pod を削除します。
```bash
kubectl delete pod -l app=sample
```

#### しばらく待ってから、再度アクセスします。
```bash
kubectl run curl --rm -it --image=curlimages/curl --restart=Never -- \
  curl -H "Host: sample.local" http://localhost
```

#### 🔍 観察ポイント
- Pod が入れ替わってもアクセス方法は変わらない
- Ingress / Service の設定は変えていない
- アプリは「外から来ている」ことを知らない

## 6. Service / Ingress / アプリの責務整理

ここで、役割を整理します。

### アプリ（Pod）
- リクエストを処理する
- 通信経路を知らない

### Service
- Pod への内部接続点
- Pod の増減を吸収する

### Ingress
- 外部からの入口
- ルーティングを定義する

通信の責務はアプリの外側に押し出されているという構造になっています。

### 7. 何を「やっていないか」が重要

このハンズオンでは、次のことを 一切やっていません。
- アプリ側の設定変更
- IP アドレスの指定
- Pod 名の意識

それでも、
- 外からアクセスでき
- Pod が入れ替わっても動く
という状態が成立しています。

### まとめ：このハンズオンで確認したこと

このハンズオンで体感したのは、Ingress は通信を便利にする仕組みではなく、責務を分離する仕組みという点です。
- アプリは処理に集中する
- Service は内部接続を担う
- Ingress は外部との境界を担う

この分離があるからこそ、
- 構成を変えられる
- Pod を捨てられる
- スケールできる

というクラウドネイティブな設計が成立しています。
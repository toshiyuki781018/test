# 問題１６：ConfigMapのTLS通信無効化

## 【▼回答】

#### ▼指定のconfigMapからTLS1.2削除（削除前）
```yaml
kubectl edit configmap -n nginx-static ngnix-config

apiVersion: v1
data:
  nginx.conf: |
    events {}
    http {
      server {
        listen 443 ssl;
        ssl_protocols TLSv1.2 TLSv1.3;  #この行にあるTLS1.2の削除を行う
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;

        location / {
          return 200 "Hello from NGINX with TLS!\n";
        }
      }
    }
kind: ConfigMap
```

#### ▼指定のconfigMapからTLS1.2削除（削除後）
```yaml
apiVersion: v1
data:
  nginx.conf: |
    events {}
    http {
      server {
        listen 443 ssl;
        ssl_protocols TLSv1.3; # TLS1.2の削除を実施
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;

        location / {
          return 200 "Hello from NGINX with TLS!\n";
        }
      }
    }
kind: ConfigMap
```


#### ▼kubectl rollout restartコマンドの実行
```bash
kubectl rollout restart deployment -n nginx-static nginx-static 
```

#### ▼curlコマンドを使用した疎通確認
```bash
curl --tls-max 1.2 https://web.k8s.local -k
```

```
curl: (35) OpenSSL/3.0.13: error:0A00042E:SSL routines::tlsv1 alert protocol version
※このコマンドではないかもけど、何かしらのエラーが帰ってくれば成功になります。
```

## ★解説
```
2025/3の末時点ではこのような問題形式で記載されていましたが、今はTLS1.3通信のみになりますので、TLS1.2を追加しなさいとの問題
なっている可能性があるとのことです。
その場合にはここに記載されたものと反対の手順を行えばよいので、肝心なことは変更後に「kubectl rollout restart」コマンドを実行して
設定を適用する必要があるので、そこを注意するようにしましょう

```

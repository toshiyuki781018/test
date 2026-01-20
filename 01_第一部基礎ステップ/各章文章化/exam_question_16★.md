# 問題１６：ConfigMapのTLS通信無効化

## 【問題】

k8sのDeploymentはTLSが有効になっているWebサーバを利用している。サポートしているTLSはTLSv1.2とTLSv1.3の２つになっており
これをTLSv1.3のサポートのみの設定を行い、最後にCurlコマンドを使用した通信の確認を実施

curl --tls-max 1.2 https://web.k8s.local -k

TLS通信の対象となるDeployment、ConfigMap、NameSpaceは以下に記載



## 【設定情報】

- Deployment `nginx-static`
- Namespace `nginx-static`
- ConfigMap `nginx-config`


## 【■事前準備】

#### ■NameSpaceの作成
```bash
kubectl create ns nginx-static
```

#### ■TLSを使用するためのSecretの作成
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=web.k8s.local/O=web.k8s.local"
kubectl create secret tls nginx-tls --cert=tls.crt --key=tls.key -n nginx-static
```

#### ■TLSを登録したConfigMapの作成とDeployment＆Serviceの作成
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
data:
  nginx.conf: |
    events {}
    http {
      server {
        listen 443 ssl;
        ssl_protocols TLSv1.2 TLSv1.3; # initial state allowing TLS 1.2 and 1.3
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;
        
        location / {
          return 200 "Hello from NGINX with TLS!\n";
        }
      }
    }
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: nginx-static
EOF
```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-static
  namespace: nginx-static
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-static
  template:
    metadata:
      labels:
        app: nginx-static
    spec:
      volumes:
      - name: config
        configMap:
          name: nginx-config
      - name: nginx-tls
        secret:
          secretName: nginx-tls # Assuming the TLS secret is pre-created
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 443
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: nginx-tls
          mountPath: /etc/nginx/ssl
EOF
```

```bash
kubectl expose deployment -n nginx-static nginx-static --port=443 --target-port=443 --name=nginx-svc --type=NodePort
```

#### ■作成したServiceのClusterIPを確認して、ホスト名に反映させる
```bash
kubectl get svc -n nginx-static

NAME         TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
nginx-svc   NodePort   10.102.100.251   <none>        443:30384/TCP   27m

vim /etc/hosts
10.102.100.251 web.k8s.local
```

#### ■疎通確認（CurlコマンドでTLSv1.2の疎通確認）
```bash
curl --tls-max 1.2 https://web.k8s.local -k

Hello from NGINX with TLS!
```

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
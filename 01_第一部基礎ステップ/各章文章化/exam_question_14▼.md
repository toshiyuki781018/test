# 問題１４：トラブルシューティング（EtcdのサーバーIP設定とCoreDNSの設定）


## 【▼回答】

#### ▼kube-apiserverへの通信回復設定
```
etcdが今まで外部にあって、内部の変更になったということからkube-apiserverに記載されているetcdのIPアドレスに問題ある可能性が高い
```

```bash
vim /etc/kubernetes/manifests/kube-apiserver.yaml

呉：- --etcd-servers=http://192.168.xxx.xxx:2379
正：- --etcd-servers=http://127.0.0.1:2379
```

```
etcdサーバーのIPアドレスが内部ではなく、外部に向いている（正確にはローカル環境の他の箇所）のでそれを変更すればOK
```

```bash
systemctl deamon-reload && systemctl restart kubelet

kubectl get node
```

#### ▼kube-schedullerのトラブルシューティング
```
2025/5/26現在で回答方法は不明です

```

## ★解説
```
ここでは、etcdサーバーのIPアドレスを変更する問題についてはなんとかなりますが、その後kube-schedulerのトラブルシューティングの
回答方法がまだ不明です。これはわかり次第、回答方法をこちらに記載するようにします。

```


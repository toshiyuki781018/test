# 問題２：cri-dockerdのインストールとIPV4よるのパケット転送の有効化

#### ■参考となるk8sドキュメント
> [Killacoda](https://killercoda.com/playgrounds/scenario/kubernetes)

> [IPV4ポートフォワーディングの有効化](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional)

> [systemパラメータの構成](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional)

## ▼回答

#### ▼cri-dockerをdpkgコマンドを使用してのインストール
```bash
dpkg -i ~/cri-dockerd_0.3.9.3-0.ubuntu-focal_amd64.deb
```

```
Selecting previously unselected package cri-dockerd.
(Reading database ... 140092 files and directories currently installed.)
Preparing to unpack .../cri-dockerd_0.3.9.3-0.ubuntu-focal_amd64.deb ...
Unpacking cri-dockerd (0.3.9~3-0~ubuntu-focal) ...
Setting up cri-dockerd (0.3.9~3-0~ubuntu-focal) ...
Created symlink /etc/systemd/system/multi-user.target.wants/cri-docker.service → /usr/lib/systemd/system/cri-docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/cri-docker.socket → /usr/lib/systemd/system/cri-docker.socket.
※上記概要が表記されていたら成功
```

#### ▼cri-dockerの起動（ここではインストールを実施した「cri-dockerd」ではなく「cri-docker.service」とする必要があるので注意）
```bash
systemctl start cri-docker.service
systemctl enable cri-docker.service
systemctl status cri-docker.service
```

```
● cri-docker.service - CRI Interface for Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/cri-docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-04-07 01:10:20 UTC; 4min 35s ago
※Activeになっていたら動作しています。
```

#### ▼IPv4によるパケット転送の有効化）
```bash
vim /etc/sysctl.d/k8s.conf
```

```conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
※vimで上記3つ（本番時は４つ）の記載を行い保存をする
```

```bash
sudo sysctl --system

net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
※追記した3つが記載されていればOK
```

```
■回答の考察
ここでは、dpkgコマンドを用いてのインストールと、IPV4によるパケット通信の有効化ができれば問題としてはそこまで難しくないです。
実際のCKAでは「cri-dockerd_0.3.9.3-0.ubuntu-fotal_amd64.deb」はすでに環境内にあります。
cri-dockerインストール後は
・cri-dockerを起動する（Start）
・cri-dockerのSystemdを有効化にして、開始する必要があり（enableの適用）があるのでそこを注意
その後は、ブリッジネットワークを設定すれば完了となります。
```



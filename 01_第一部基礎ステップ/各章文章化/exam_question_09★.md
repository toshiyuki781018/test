# 問題９：FlannelとCalicoの選択

#### ■参考となるドキュメント

> [calicoのCRDダウンロード](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico)



## 【▼回答】

#### ▼Calicoのインストール:この問題にて使用するCNIプラグインはCalicoになりますので、Calicoのインストールを実施します。

```bash
kubectl apply -f  https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
```

#### ▼CRDダウンロード
```bash
wget -o- https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/custom-resources.yaml
```
```
デフォルト設定におけるCalicoのCRDには環境に合わないPodのCIDRが登録されているので、
別途ダウンロードを行い、PodのCIDRを編集しなおす必要がある。その対象となるドキュメントは参考となるドキュメントにURLが
記載されているが、実際の試験ではその場所を探して、ダウンロードを行う必要あり
```

#### ▼CIDRの確認
```bash
ps aux | grep kube-controller-manager
```

```
etes/controller-manager.conf --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf --bind-address=127.0.0.1 \
--client-ca-file=/etc/kubernetes/pki/ca.crt --cluster-cidr=10.244.0.0/16 ~~~~

この「--cluster-cidr=10.244.0.0/16」を、「custom-resources.yaml」に適用を行う必要がある

```

#### ▼「custom-resources.yaml」のCIDRの変更と適用（※「10.244.0.0/16」に変更、適用を行う）
```yaml
vim custom-resources.yaml

apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 192.168.1.0/16 #「10.244.0.0/16」に変更
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()


kubectl apply -f custom-resources.yaml
```

## ★解説

- 2つのCNIプラグインをインストールする問題になりますが、NetworkPocicyを使用できるかが、焦点になり、Calicoが当てはまるのでインストールを実施
- その後、CoreDNSにエラーが発生するが、原因はCRDに記載されているPodのCIDRがk8s環境のものと適合しないので発生

- この問題は一見すると、URLにあるYamlファイルを適用すればよいように見えるが、適用するだけどCoreDNSが動作しなくなってしまう
- CoreDNSを動作させるには、custom-resourcesを編集して適用する必要があるのでそこを注意すればよし


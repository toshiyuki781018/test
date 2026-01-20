# 問題１：HorizontalPodAutoscaler(hpa)の設定

## 【問題】

対象となるNamespaceに作成されているDeploymentに対し、HPA（Horizontal Pod Autoscaler）を作成し,
下記設定情報を元にHPAのYamlファイルを作成し、対象のKubernetesクラスターへApplyしなさい



## 【設定情報】

- Namespace   `autoscale`
- HPAの名称：   `apache-server`
- Deployment名 `apache-server`
- Pod1つあたりCPU使用率 `50％`
- 最小レプリカ数 `1`
- 最大レプリカ数 `4`　
- 安定化ウィンドウの設定秒数 `30`
- 他設定 `stabilizationWindow`

## 【■事前準備】

#### ■NameSpaceとDeploymentの作成

```bash
kubectl create ns autoscale
kubectl create deployment apache-server --image=nginx --replicas=1 -n autoscale
```

#### ■参考となるk8sドキュメント

> [HPAのテンプレートYamlファイル](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)

> [安定化ウィンドウの設定](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#stabilization-window)


## 【▼回答】

> [リンク先のドキュメントへのアクセスして、ベースのYamlファイルを転記<Status以降のものはいらない>](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)

> [安定化ウィンドウの転記（リンク先のYamlファイルに記載する概要を転記>](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#stabilization-window)


#### ▼APIリソースでApiversionを確認する（テンプレートに記載されているVerとk8sに登録されているVerに違いがある可能性があるため

```bash
kubectl api-reources | grep hpa

NAME                                SHORTNAMES       APIVERSION                          NAMESPACED   KIND
horizontalpodautoscalers            hpa              autoscaling/v2                      true         HorizontalPodAutoscaler
```


#### ▼２つを組み合わせて下記のYamlファイルを作成する（apiVersionが「v2」でないとApplyできないかもしれないので注意）


```bash
vim hpa.yaml
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: apache-server
  namespace: autoscale
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: apache-server
  minReplicas: 1
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 30
```

#### ▼対象のYamlファイルを作成したら、Applyを実施する


```bash
kubectl apply -f hpa.yaml
```

## ●作成した結果


```bash
kubectl get hpa -n autoscale

NAME            REFERENCE                  TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
apache-server   Deployment/apache-server   <unknown>/50%   1         4         1          16s

kubectl describe hpa -n autoscale

Name:                                                  apache-server
Namespace:                                             autoscale
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Tue, 25 Mar 2025 01:02:44 +0000
Reference:                                             Deployment/apache-server
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  <unknown> / 50%
Min replicas:                                          1
Max replicas:                                          4
Behavior:
  Scale Up:
    Stabilization Window: 0 seconds
    Select Policy: Max
    Policies:
      - Type: Pods     Value: 4    Period: 15 seconds
      - Type: Percent  Value: 100  Period: 15 seconds
  Scale Down:
    Stabilization Window: 30 seconds
    Select Policy: Max
    Policies:
      - Type: Percent  Value: 100  Period: 15 seconds


```


## ★解説

- この問題は対象となるDeploymentにHPAを適用する問題になります。
- k8sのドキュメントはHPAを検索すれば表記可能
- その上で「behavior:」以下を追加する必要があり、そのドキュメントを別途探して記載する必要があるか、覚えておく必要がある。
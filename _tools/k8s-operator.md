---
layout: default
title: OpenSearch Kubernetes操作
nav_order: 80
has_children: false
---

OpenSearch Kubernetes操作员是开放的-来源Kubernetes操作员，可以帮助自动化集装箱环境中OpenSearch和OpenSearch仪表板的部署和配置。操作员可以管理多个OpenSearch群集，这些集群可以根据您的需求来上下扩展。


## 安装

运营商有两种方法：

- [使用掌舵图](#use-a-helm-chart)。
- [使用本地安装](#use-a-local-installation)。

### 使用掌舵图

如果您使用Helm管理Kubernetes群集，则可以使用OpenSearch Kubernetes操作员的云本机计算基础（CNCF）项目存储在Artifact Hub中的项目，Web-基于查找，安装和发布CNCF软件包的应用程序。

首先，登录到您的kubernetes群集，然后从[人工枢纽](https://artifacthub.io/packages/helm/opensearch-operator/opensearch-operator/)。

```
helm repo add opensearch-operator https://opster.github.io/opensearch-k8s-operator/
```

确保存储库包含在您的kubernetes群集中。

```
helm repo list | grep opensearch
```

这俩`opensearch` 和`opensearch-operator` 存储库出现在存储库列表中。


安装操作所有OpenSearch Kubernetes操作员的操作的管理器。

```
helm install opensearch-operator opensearch-operator/opensearch-operator
```

安装完成后，操作员将返回有关部署的信息`STATUS: deployed`。那么您可以配置并开始[OpenSearch集群](#deploy-a-new-opensearch-cluster)。

### 使用本地安装

如果要在现有机器上创建新的Kubernetes群集，请使用本地安装。

如果这是您第一次运行kubernetes，并且您打算在笔记本电脑上运行这些说明，请确保已安装以下内容：

- [Kubernetes](https://kubernetes.io/docs/tasks/tools/) 
- [Docker](https://docs.docker.com/engine/install/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)

在贯穿安装步骤之前，请确保您的Kubernetes环境在本地运行。使用Minikube时，打开一个新的终端窗口并输入`minikube start`。Kubernetes现在将使用一个名称的命名空间的容器化Minikube群集`default`。

然后使用以下步骤安装OpenSearch Kubernetes操作员：

1. 在您的首选目录中，克隆[OpenSearch Kubernetes运营商存储库](https://github.com/Opster/opensearch-k8s-operator)。使用使用`cd`。
2. 去`opensearch-operator` 文件夹。
3. 进入`make build manifests`。
4. 开始一个Kubernetes群集。使用Minikube时，打开一个新的终端窗口并输入`minikube start`。Kubernetes现在将使用一个名称的命名空间的容器化Minikube群集`default`。确保这一点`~/.kube/config` 指向群集。

```yml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/naarcha/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Mon, 29 Aug 2022 10:11:47 CDT
        provider: minikube.sigs.k8s.io
        version: v1.26.1
      name: cluster_info
    server: https://127.0.0.1:61661
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 29 Aug 2022 10:11:47 CDT
        provider: minikube.sigs.k8s.io
        version: v1.26.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/naarcha/.minikube/profiles/minikube/client.crt
    client-key: /Users/naarcha/.minikube/profiles/minikube/client.key
```    
   
5. 进入`make install` 为了创建在Kubernetes群集中运行的CustomResourcercedefinition。
6. 启动OpenSearch Kubernetes操作员。进入`make run`。

## 验证Kubernetes部署

为了确保Kubernetes将OpenSearch Kubernetes操作员视为名称空间，请输入`k get ns | grep opensearch`。两个都`opensearch` 和`opensearch-operator-system` 应该出现`Active`。

使用操作员活动，请使用`k get pod -n opensearch-operator-system` 确保操作员的豆荚正在运行。

```
NAME                                              READY   STATUS   RESTARTS   AGE
opensearch-operator-controller-manager-<pod-id>   2/2     Running  0          25m
```

在运行Kubernetes群集后，您现在可以在集群中运行OpenSearch。

## 部署新的OpenSearch集群

从您的克隆opensearch kubernetes操作员存储库中，导航到`opensearch-operator/examples` 目录。在那里你会发现`opensearch-cluster.yaml` 文件，可以根据群集的需求进行自定义，包括`clusterName` 这是您新的OpenSearch集群将驻留的名称空间。

通过配置群集，运行`kubectl apply` 命令。

```
kubectl apply -f opensearch-cluster.yaml
```

操作员创建了多个POD，包括一个bootstrap Pod，三个OpenSearch集群POD和一个仪表板Pod。要连接到您的群集，请使用`port-forward` 命令。

```
kubectl port-forward svc/my-cluster-dashboards 5601
```

打开http：// localhost：5601在您的首选浏览器中，并使用默认演示凭据登录`admin / admin`。您还可以通过转发到端口9200来针对OpenSearch REST API运行Curl命令。

```
kubectl port-forward svc/my-cluster 9200
```

为了删除OpenSearch集群，请删除集群资源。以下命令删除了集群名称空间及其所有资源。

```
kubectl delete -f opensearch-cluster.yaml
```

## 下一步

要了解有关如何自定义Kubernetes OpenSearch集群的更多信息，包括数据持久性，身份验证方法和缩放，请参见[OpenSearch Kubernetes操作员用户指南](https://github.com/Opster/opensearch-k8s-operator/blob/main/docs/userguide/main.md)。

如果您想为OpenSearch Kubernetes操作员的开发做出贡献，请参阅回购[设计文件](https://github.com/Opster/opensearch-k8s-operator/blob/main/docs/designs/high-level.md)


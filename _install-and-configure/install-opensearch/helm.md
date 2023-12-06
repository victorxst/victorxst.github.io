---
layout: default
title: Helm
parent: 安装 OpenSearch
nav_order: 6
redirect_from:
  - /opensearch/install/helm/
---

# 掌舵

Helm 是一个软件包管理器，可让你在 Kubernetes 集群中轻松安装和管理 OpenSearch。你可以在 YAML 文件中定义 OpenSearch 配置，并使用 Helm 以版本控制且可重现的方式部署应用程序。

Helm 图表包含下表中描述的资源。

资源 | 描述
:--- | :---
 `Chart.yaml` | 有关图表的信息。
 `values.yaml` | 图表的默认配置值。
 `templates` | 与值组合以生成 Kubernetes 清单文件的模板。

默认 Helm 图表中的规范支持许多标准用例和设置。你可以修改默认图表以配置所需的规范，并设置传输层安全性（TLS）和基于角色的访问控制（RBAC）。

有关缺省配置、配置安全性的步骤和可配置参数的信息，请参见。[README](https://github.com/opensearch-project/helm-charts/blob/main/README.md)

此处的说明假设你有一个预安装了 Helm 的 Kubernetes 集群。[Kubernetes 文档](https://kubernetes.io/docs/setup/)有关配置 Kubernetes 集群的步骤和[Helm 文档](https://helm.sh/docs/intro/install/)安装 Helm 的步骤，请参阅。
{:.note}

## 先决条件

默认的 Helm 图表部署一个三节点集群。我们建议你至少为此部署准备 8 GiB 的内存。例如，如果可用内存少于 4 GiB，则部署可能会失败。

## 使用 Helm 安装 OpenSearch

1. 将存储库添加到 `opensearch` [helm-charts](https://github.com/opensearch-project/helm-charts) Helm：

   ```bash
   helm repo add opensearch https://opensearch-project.github.io/helm-charts/
   ```
   {% include copy.html %}

1. 从图表存储库本地更新可用图表：

   ```bash
   helm repo update
   ```
   {% include copy.html %}

1. 要搜索与 OpenSearch 相关的 Helm 图表，请执行以下操作：

   ```bash
   helm search repo opensearch
   ```
   {% include copy.html %}

   ```bash
   NAME                            	CHART VERSION	APP VERSION	DESCRIPTION                           
   opensearch/opensearch           	1.0.7        	1.0.0      	A Helm chart for OpenSearch           
   opensearch/opensearch-dashboards	1.0.4        	1.0.0      	A Helm chart for OpenSearch Dashboards
   ```

1. 部署 OpenSearch：

   ```bash
   helm install my-deployment opensearch/opensearch
   ```
   {% include copy.html %}

你也可以手动构建 `opensearch-1.0.0.tgz` 文件：

1. 切换到 `opensearch` 目录：

   ```bash
   cd charts/opensearch
   ```
   {% include copy.html %}

1. 打包 Helm 图表：

   ```bash
   helm package .
   ```
   {% include copy.html %}

1. 部署 OpenSearch：

   ```bash
   helm install --generate-name opensearch-1.0.0.tgz
   ```
   {% include copy.html %}

   输出显示从安装中实例化的规范。若要自定义部署，请传入要使用自定义 YAML 文件覆盖的值：

   ```bash
   helm install --values=customvalues.yaml opensearch-1.0.0.tgz
   ```
   {% include copy.html %}

#### 示例输出

  ```yaml
  NAME: opensearch-1-1629223146
  LAST DEPLOYED: Tue Aug 17 17:59:07 2021
  NAMESPACE: default
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  Watch all cluster members come up.
    $ kubectl get pods --namespace=default -l app=opensearch-cluster-master -w
  ```

要确保你的 OpenSearch Pod 已启动并运行，请运行以下命令：

```bash
$ kubectl get pods
NAME                                                  READY   STATUS    RESTARTS   AGE
opensearch-cluster-master-0                           1/1     Running   0          3m56s
opensearch-cluster-master-1                           1/1     Running   0          3m56s
opensearch-cluster-master-2                           1/1     Running   0          3m56s
```

要访问 OpenSearch shell，请执行以下操作：

```bash
$ kubectl exec -it opensearch-cluster-master-0 -- /bin/bash
```
{% include copy.html %}

你可以向 Pod 发送请求，以验证 OpenSearch 是否已启动并正在运行：

```json
$ curl -XGET https://localhost:9200 -u 'admin:admin' --insecure
{
  "name" : "opensearch-cluster-master-1",
  "cluster_name" : "opensearch-cluster",
  "cluster_uuid" : "hP2gq5bPS3SLp8Z7wXm8YQ",
  "version" : {
    "distribution" : "opensearch",
    "number" : <version>,
    "build_type" : <build-type>,
    "build_hash" : <build-hash>,
    "build_date" : <build-date>,
    "build_snapshot" : false,
    "lucene_version" : <lucene-version>,
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

## 使用 Helm 卸载

要确定要删除的 OpenSearch 部署，请执行以下操作：

```bash
$ helm list
NAME  NAMESPACEREVISIONUPDATED  STATUS  CHART APP VERSION
opensearch-1-1629223146 default 1 2021-08-17 17:59:07.664498239 +0000 UTCdeployedopensearch-1.0.0    1.0.0       
```

若要删除或卸载部署，请运行以下命令：

```bash
helm delete opensearch-1-1629223146
```
{% include copy.html %}

有关安装 OpenSearch 控制面板的步骤，请参阅[用于安装 OpenSearch 控制面板的 Helm]({{site.url}}{{site.baseurl}}/dashboards/install/helm/)。
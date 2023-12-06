---
layout: default
title: ML Commons集群设置
has_children: false
nav_order: 10
---

# ML Commons集群设置


为了增强和自定义用于机器学习的OpenSearch集群（ML），您可以在“ OpenSearch.yml”文件中为ML Commons插件添加并修改多个配置设置。

要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## ML节点

默认情况下，ML任务和模型仅在ML节点上运行。当配置没有`data` 节点角色，ML节点不存储任何碎片，而是在运行时计算资源需求。要使用ML节点，请在您的`opensearch.yml` 文件。给您的节点一个自定义名称，并将节点角色定义为`ml`：

```yml
node.roles: [ ml ]
```

## 仅在ML节点上运行任务和模型

如果`true`，ML Commons任务和模型仅在ML节点上运行ML任务。如果`false`，任务和模型首先在ML节点上运行。如果不存在ML节点，则在数据节点上运行任务和模型。

我们建议设置`plugins.ml_commons.only_run_on_ml_node` 到`true` 在生产集群上。
{:.tip}


### 环境

```
plugins.ml_commons.only_run_on_ml_node: true
```

### 值

- 默认值：`true`
- 价值范围：`true` 或者`false`

## 调度任务到ML节点

`round_robin` 使用循环路由将ML任务分配给ML节点。`least_load` 收集来自所有ML节点的运行时信息，例如JVM HEAP内存使用和运行任务，然后将任务分配给最低负载的ML节点。


### 环境

```
plugins.ml_commons.task_dispatch_policy: round_robin
```


### 值

- 默认值：`round_robin`
- 价值范围：`round_robin` 或者`least_load`

## 设置每个节点的ML任务数

设置可以在每个ML节点上运行的ML任务数。设置为`0`，任何节点上没有ML任务。

### 环境

```
plugins.ml_commons.max_ml_task_per_node: 10
```

### 值

- 默认值：`10`
- 价值范围：[0，10,000]

## 设置每个节点的ML型号

设置可以部署到每个ML节点的ML模型的数量。设置为`0`，没有ML模型可以在任何节点上部署。

### 环境

```
plugins.ml_commons.max_model_on_node: 10
```

### 值

- 默认值：`10`
- 价值范围：[0，10,000]

## 设置同步工作间隔

返回运行时信息时[配置文件API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/profile/)，ML Commons将执行常规工作，以在每个节点上同步新部署或未剥削的模型。设置为`0`，ML Commons立即停止同步-工作。


### 环境

```
plugins.ml_commons.sync_up_job_interval_in_seconds: 3
```

### 值

- 默认值：`3`
- 值范围：[0，86,400]

## 预测监视请求

控制一个节点上监视多少预测请求。如果设置为`0`，OpenSearch清除所有监视预测缓存中的请求，并且不会监视新的预测请求。

### 环境

```
plugins.ml_commons.monitoring_request_count: 100
```

### 价值范围

- 默认值：`100`
- 价值范围：[0，10,000,000]

## 每个节点的注册模型任务

控制一个可以在一个节点上并行运行的寄存器模型任务。如果设置为`0`，您无法在任何节点上运行注册模型任务。

### 环境

```
plugins.ml_commons.max_register_model_tasks_per_node: 10
```


### 值

- 默认值：`10`
- 价值范围：[0，10]


## 每个节点部署模型任务

控制一个节点可以并行运行多少个部署模型任务。如果设置为0，则无法将模型部署到任何节点。

### 环境

```
plugins.ml_commons.max_deploy_model_tasks_per_node: 10
```

### 值

- 默认值：`10`
- 价值范围：[0，10]

## 使用URL注册模型

此设置使您能够使用URL注册模型。默认情况下，ML Commons仅允许注册[预估计]({{site.url}}{{site.baseurl}}//ml-commons-plugin/pretrained-models/) 来自OpenSearch模型存储库中的模型。

### 环境

```
plugins.ml_commons.allow_registering_model_via_url: false
```

### 值

- 默认值：false
- 有效值：`false`，`true`

## 使用本地文件注册模型

此设置使您能够使用本地文件注册模型。默认情况下，ML Commons仅允许注册[预估计]({{site.url}}{{site.baseurl}}//ml-commons-plugin/pretrained-models/) 来自OpenSearch模型存储库中的模型。

### 环境

```
plugins.ml_commons.allow_registering_model_via_local_file: false
```

### 值

- 默认值：false
- 有效值：`false`，`true`

## 添加可信赖的URL

默认值允许您从任何HTTP/HTTP/FTP/本地文件中注册模型文件。您可以将此值更改为限制可信赖的模型URL。


### 环境

此值得信赖的URL设置的默认URL值不安全。为了安全性，请使用您自己的正则正子字符串到包含您的模型的受信任的存储库中，例如`https://github.com/opensearch-project/ml-commons/blob/2.x/ml-algorithms/src/test/resources/org/opensearch/ml/engine/algorithms/text_embedding/*`。
{: .warning }


```
plugins.ml_commons.trusted_url_regex: <model-repository-url>
```

### 值

- 默认值：`"^(https?|ftp|file)://[-a-zA-Z0-9+&@#/%?=~_|!:,.;]*[-a-zA-Z0-9+&@#/%=~_|]"`
- 值范围：Java正则表达式（REGEX）字符串

## 分配任务超时

分配ML任务将在几秒钟内进行多长时间。超时后，任务将失败。

### 环境

```
plugins.ml_commons.ml_task_timeout_in_seconds: 600
```

### 值

- 默认值：600
- 价值范围：[1，86,400]

## 设置本机内存阈值

设置断路器，该断路器在运行ML任务之前检查所有系统内存使用情况。如果本机内存超过阈值，则OpenSearch会引发异常，并停止运行任何ML任务。

值基于可用内存的百分比。设置为`0`，不会运行ML任务。设置为`100`，断路器关闭，没有阈值。

### 环境

```
plugins.ml_commons.native_memory_threshold: 90
```

### 值

- 默认值：90
- 价值范围：[0，100]

## 允许自定义部署计划

启用后，此设置使用户能够根据用户的权限将模型部署到特定的ML节点。

### 环境

```
plugins.ml_commons.allow_custom_deployment_plan: false
```

### 值

- 默认值：false
- 有效值：`false`，`true`

## 启用自动重新部署

此设置会在集群故障时自动重新部署或部分部署模型。如果集群崩溃中的所有ML节点，则模型切换到`DEPLOYED_FAILED` 状态，模型必须手动部署。

### 环境

```
plugins.ml_commons.model_auto_redeploy.enable: false
```

### 值

- 默认值：false
- 有效值：`false`，`true`

## 为汽车重新部署设置退休

此设置设置了部署或部分部署模型的次数的限制，当群集失败或新的ML节点中的ML节点加入群集时，将尝试重新部署。

### 环境

```
plugins.ml_commons.model_auto_redeploy.lifetime_retry_times: 3
```

### 值

- 默认值：3
- 价值范围：[0，100]

## 设置自动重新部署成功率

此设置设置了自动的成功之比-基于群集中可用的ML节点的模型重新部署。例如，如果ML节点在集群中崩溃，则自动还原协议会添加另一个节点或退休崩溃的节点。如果比率为`0.7` 和所有ML节点的70％成功重新部署了该模型-重新部署激活，重新部署是成功的。如果模型重新部署在不到70％的可用ML节点上，则自动-重新部署重新恢复直到重新部署成功或开放搜索到达[最大恢复次数](#set-retires-for-auto-redeploy)。

### 环境

```
plugins.ml_commons.model_auto_redeploy_success_ratio: 0.8
```

### 值

- 默认值：0.8
- 值范围：[0，1]

## 运行Python-基于模型

设置为`true`，此设置使能够运行Python-由OpenSearch支持的基于模型，例如[指标相关]({{site.url}}{{site.baseurl}}/ml-commons-plugin/algorithms/#metrics-correlation)。

### 环境

```
plugins.ml_commons.enable_inhouse_python_model: false
```

### 值

- 默认值：false
- 有效值：`false`，`true`

## 启用连接器的访问控制

设置为`true`，该设置允许管理员使用使用的访问和权限`backend_roles`。

### 环境

```
plugins.ml_commons.connector_access_control_enabled: true
```

### 值

- 默认值：false
- 有效值：`false`，`true`







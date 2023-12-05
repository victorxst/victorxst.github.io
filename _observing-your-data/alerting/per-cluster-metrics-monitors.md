---
layout: default
title: 每个集群指标监视器
nav_order: 15
parent: 监视器
grand_parent: 警报
has_children: false
---

# 每个集群指标监视器

每个群集指标监视器是一种警报监视器，可从单个集群中收集和分析指标，从而提供了群集的性能和健康的见解。您可以设置警报以监视某些条件，例如何时：

- 簇健康达到黄色或红色状态。
- 簇-水平指标---例如，CPU用法和JVM内存使用率---达到指定的阈值。
- 节点-水平指标---例如，可用的磁盘空间，JVM内存使用情况和CPU使用情况---达到指定的阈值。
- 存储的文档总数达到指定的阈值。

## 创建集群指标监视器

要创建群集指标监视器，请按照以下步骤：

1. 选择**警报** >**监视器** >**创建监视器**。
2. 选择**每个群集指标监视器** 选项。
3. 在查询部分中，选择**请求类型** 从下拉列表中。
4. （可选）如果要过滤API响应以仅使用某些路径参数，请在下面输入这些参数**查询参数**。大多数可用于监视群集状态支持路径参数的API如其文档中所述（例如，逗号-索引名称的分开列表）。
5. 在里面[触发器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/triggers/) 部分，指示哪些条件将触发警报。触发条件自动填充`painless ctx` 多变的。例如，观看集群统计数据的群集监视器使用触发条件`ctx.results[0].indices.count <= 0`，这会根据查询返回的索引数量触发警报。为了获得更多特异性，请添加API支持的任何其他无痛条件。要查看条件响应的示例，请选择**预览条件响应**。
6. 在“操作”部分中，指示如何在满足触发条件时如何通知用户。
7. 选择**创造**。您的新显示器出现在**监视器** 列表。

以下示例显示了集群指标监视器的配置。

<img src="{{site.url}}{{site.baseurl}}/images/cluster-metrics.png" alt="Cluster metrics monitor" width="700"/>

## 支持的API

触发条件使用以下API端点的响应。大多数可用于监视群集状态支持路径参数的API（例如，逗号-索引名称的分开列表）。他们不支持查询参数。

- [_CLUSTER/HEALTY]({{site.url}}{{site.baseurl}}/api-reference/cluster-health/)
- [_ cluster/stats]({{site.url}}{{site.baseurl}}/api-reference/cluster-stats/)
- [_ cluster/设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-settings/)
- [_ nodes/stats]({{site.url}}{{site.baseurl}}/opensearch/popular-api/#get-node-statistics)
- [_猫/索引]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-indices/)
- [_cat/pending_tasks]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-pending-tasks/)
- [_猫/恢复]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-recovery/)
- [_猫/碎片]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-shards/)
- [_猫/快照]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-snapshots/)
- [_猫/任务]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-tasks/)

## 限制API字段

如果您想从API响应中隐藏字段，而不是公开它们以提醒，请重新配置[supported_json_payloads.json](https://github.com/opensearch-project/alerting/blob/main/alerting/src/main/resources/org/opensearch/alerting/settings/supported_json_payloads.json) 在警报插件中文件。该文件作为您要在警报中使用的API字段的允许列表。默认情况下，所有API及其参数可用于监视和触发条件。

但是，您可以修改文件，以便只能为引用的API创建群集公制显示器。此外，只有受支持文件中引用的字段才能创建触发条件。这`supported_json_payloads.json` 允许创建一个群集指标监视器`_cluster/stats` API和触发条件`indices.shards.total` 和`indices.shards.index.shards.min` 字段。

```json
"/_cluster/stats": {
  "indices": [
    "shards.total",
    "shards.index.shards.min"
  ]
}
```

## 无痛的触发因素

无痛的脚本定义了用于群集指标监视的触发器，类似于每个查询或每个桶形监视器，这些触发器是使用提取查询定义选项定义的。无痛脚本由至少一个语句和您希望运行的任何其他功能组成。

集群指标监视器支持**十** 触发器。

在下面的示例中，JSON对象会创建一个触发器，该触发器在群集健康为黄色时发送警报。`script` 指向`source` 到无痛剧本`ctx.results[0].status == \"yellow\`。

```json
{
  "name": "Cluster Health Monitor",
  "type": "monitor",
  "monitor_type": "query_level_monitor",
  "enabled": true,
  "schedule": {
    "period": {
      "unit": "MINUTES",
      "interval": 1
    }
  },
  "inputs": [
    {
      "uri": {
        "api_type": "CLUSTER_HEALTH",
        "path": "_cluster/health/",
        "path_params": "",
        "url": "http://localhost:9200/_cluster/health/"
      }
    }
  ],
  "triggers": [
    {
      "query_level_trigger": {
        "id": "Tf_L_nwBti6R6Bm-18qC",
        "name": "Yellow status trigger",
        "severity": "1",
        "condition": {
          "script": {
            "source": "ctx.results[0].status == \"yellow\"",
            "lang": "painless"
          }
        },
        "actions": []
      }
    }
  ]
}
```

看[触发变量]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/triggers/#trigger-variables) 更多`painless ctx` 可变选项。

### 限制

每个群集指标监视器具有以下限制：

- 您不能为远程簇创建显示器。
- OpenSearch集群必须处于可以监视索引条件并可以针对索引执行操作的状态。
- 从用户中删除资源权限不会阻止用户对该资源的先前监视器执行。
- 具有创建监视器的权限的用户不会阻止为其没有权限的资源创建监视器；但是，这些监视器不会运行。


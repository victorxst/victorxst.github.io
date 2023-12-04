---
layout: default
title: REST API参考
nav_order: 1
has_toc: false
has_children: true
nav_exclude: true
redirect_from:
  - /opensearch/rest-api/index/
---

# REST API参考
**引入1.0**
{: .label .label-purple }

您可以将REST API用于OpenSearch中的大多数操作。在此引用中，我们提供了API的描述，以及包括路径和HTTP方法，支持参数以及示例请求和响应的详细信息。

该引用包括OpenSearch支持的REST API。如果缺少REST API，请提供反馈或在GitHub中提交拉动请求。
{: .tip }

## 相关文章

- [分析API]({{site.url}}{{site.baseurl}}/api-reference/analyze-apis/)
- [访问控制API]({{site.url}}{{site.baseurl}}/security/access-control/api/)
- [提醒API]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/api/)
- [异常检测API]({{site.url}}{{site.baseurl}}/observing-your-data/ad/api/) 
- [猫API]({{site.url}}{{site.baseurl}}/api-reference/cat/index/)
- [群集API]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/index/)
- [常见的休息参数]({{site.url}}{{site.baseurl}}/api-reference/common-parameters/)
- [数数]({{site.url}}{{site.baseurl}}/api-reference/count/)
- [叉-集群复制API]({{site.url}}{{site.baseurl}}/tuning-your-cluster/replication-plugin/api/)
- [文档API]({{site.url}}{{site.baseurl}}/api-reference/document-apis/index/)
- [解释]({{site.url}}{{site.baseurl}}/api-reference/explain/)
- [索引API]({{site.url}}{{site.baseurl}}/api-reference/index-apis/index/)
- [索引汇总API]({{site.url}}{{site.baseurl}}/im-plugin/index-rollups/rollup-api/)
- [索引状态管理API]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/)
- [ISM错误预防API]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/api/)
- [摄入的API]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/)
- [k-NN插件API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api/)
- [ML Commons API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/) 
- [并发-搜索]({{site.url}}{{site.baseurl}}/api-reference/multi-search/)
- [节点API]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/)
- [通知API]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/api/)
- [性能分析仪API]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/pa/api/)
- [时间点API]({{site.url}}{{site.baseurl}}/search-plugins/point-in-time-api/)
- [流行的API]({{site.url}}{{site.baseurl}}/api-reference/popular-api/)
- [排名评估]({{site.url}}{{site.baseurl}}/api-reference/rank-eval/)
- [刷新搜索分析仪]({{site.url}}{{site.baseurl}}/im-plugin/refresh-analyzer/)
- [删除群集信息]({{site.url}}{{site.baseurl}}/api-reference/remote-info/)
- [根本原因分析API]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/pa/rca/api/)
- [快照管理API]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/sm-api/)
- [脚本API]({{site.url}}{{site.baseurl}}/api-reference/script-apis/index/)
- [滚动]({{site.url}}{{site.baseurl}}/api-reference/scroll/)
- [搜索]({{site.url}}{{site.baseurl}}/api-reference/search/)
- [搜索相关统计数据API]({{site.url}}{{site.baseurl}}/search-plugins/search-relevance/stats-api/)
- [安全分析API]({{site.url}}{{site.baseurl}}/security-analytics/api-tools/index/)
- [快照API]({{site.url}}{{site.baseurl}}/api-reference/snapshots/index/)
- [统计API]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/stats-api/)
- [支持单位]({{site.url}}{{site.baseurl}}/api-reference/units/)
- [任务]({{site.url}}{{site.baseurl}}/api-reference/tasks/)
- [转换API]({{site.url}}{{site.baseurl}}/im-plugin/index-transforms/transforms-apis/)





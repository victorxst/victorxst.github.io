---
layout: default
title: 远程群集信息
nav_order: 67
redirect_from: 
 - /opensearch/rest-api/remote-info/
---

# 远程群集信息
**引入1.0**
{: .label .label-purple }

此操作为您为本地群集配置的任何远程OpenSearch群集提供连接信息，例如远程群集别名，连接模式（`sniff` 或者`proxy`），种子节点的IP地址和超时设置。

响应比呼吁更全面和有用`_cluster/settings`，仅包括簇别名和种子节点。


## 路径和HTTP方法

```
GET _remote/info
```
{% include copy-curl.html %}

## 回复

```json
{
  "opensearch-cluster2": {
    "connected": true,
    "mode": "sniff",
    "seeds": [
      "172.28.0.2:9300"
    ],
    "num_nodes_connected": 1,
    "max_connections_per_cluster": 3,
    "initial_connect_timeout": "30s",
    "skip_unavailable": false
  }
}
```


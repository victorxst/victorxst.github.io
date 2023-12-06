---
layout: default
title: ISM 错误预防解决方法
parent: ISM 错误预防
grand_parent: 索引状态管理
nav_order: 5
---

# ISM 防错解决方案

以下各节列出了每个验证规则操作的错误解决方法。

---

#### 目录
1. 目录
{:toc}


---

## 索引不是写入索引

若要确认索引是否为写入索引，请运行以下请求：

```bash
GET <index>/_alias?pretty
```

如果响应不包含 `"is_write_index"`：true，则索引不是写入索引。以下示例确认索引是写入索引：

```json
{
  "<index>" : {
    "aliases" : {
      "<index_alias>" : { 
        "is_write_index" : true
      }
    }
  }
}
```

若要将索引设置为写入索引，请运行以下请求：

```bash
PUT <index>
{
  "aliases": {
    "<index_alias>" : {
        "is_write_index" : true
    }
  }
}
```

## 索引没有别名

如果索引没有别名，可以通过运行以下请求来添加一个别名：

```bash
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "<target_index>",
        "alias": "<index_alias>"
      }
    }
  ]
}
```

## 跳过滚动更新操作为 true

如果发生跳过滚动更新操作，请运行以下请求：

```bash
 GET <target_index>/_settings?pretty
```

如果在第一个示例中收到响应，则可以通过运行第二个示例中的请求来重置它：

```json
{
  "index": {
    "opendistro.index_state_management.rollover_skip": true
  }
}
```

```bash
PUT <target_index>/_settings
{
    "index": {
      "index_state_management.rollover_skip": false
  }
}
```

## 该指数已成功转存

删除以防止[索引的展期政策]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/#remove-policy-from-index)此错误再次发生。

## 滚动更新策略未 rollover_alias 索引设置

 `rollover_alias` 将索引设置添加到滚动更新策略以解决此问题。运行以下请求：

```bash
PUT _index_template/ism_rollover
{
  "index_patterns": ["<index_patterns_in_rollover_policy>"],
  "template": {
   "settings": {
    "plugins.index_state_management.rollover_alias": "<rollover_alias>"
   }
 }
}
```

## 数据过大且超出阈值

检查并[JVM 信息]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-info/)增加堆内存。

## 超出最大分片数

每个节点或每个索引的分片限制会导致此问题发生。通过运行以下请求来检查是否存在 `total_shards_per_node` 限制：

```bash
GET /_cluster/settings
```

如果响应包含 `total_shards_per_node`，则通过运行以下请求暂时增加其值：

```bash
PUT _cluster/settings
{
   "transient":{
      "cluster.routing.allocation.total_shards_per_node":100
   }
}
```

要检查索引是否存在分片限制，请运行以下请求：

```bash
GET <index>/_settings/index.routing-
```

如果响应包含第一个示例中的设置，请增加其值或将其 `-1` 设置为无限分片，如第二个示例所示：

```json
"index" : {
        "routing" : {
          "allocation" : {
            "total_shards_per_node" : "10"
          }
        }
      }
```

```bash
PUT <index>/_settings
{"index.routing.allocation.total_shards_per_node":-1}
```

## 索引是某些数据流的写入索引

如果仍要删除索引，请检查你的[数据流]({{site.url}}{{site.baseurl}}/opensearch/data-streams/)设置并更改写入索引。

## 索引被阻止

通常，索引被阻止是因为磁盘使用率已超过淹没阶段水位线，并且索引具有 `read-only-allow-delete` 阻止。要解决此问题，你可以：

1. 删除该 `-index.blocks.read_only_allow_delete-` 参数。
1. 暂时增加磁盘水印。
1. 暂时禁用磁盘分配阈值。

为了防止问题再次发生，最好通过增加磁盘空间、添加新节点或删除不再需要的数据或索引来减少磁盘的使用量。

通过运行以下请求进行删除 `-index.blocks.read_only_allow_delete-`：

```bash
PUT <index>/_settings
{
    "index.blocks.read_only_allow_delete": null
}
```

通过运行以下请求来增加低磁盘水印：

```bash
PUT _cluster/settings
{
    "transient": {
        "cluster": {
            "routing": {
                "allocation": {
                    "disk": {
                        "watermark": {
                            "low": "25.0gb"
                        }
                    }
                }
            }
        }
    }
}
```

通过运行以下请求来禁用磁盘分配阈值：

```bash
PUT _cluster/settings
{
    "transient": {
        "cluster": {
            "routing": {
                "allocation": {
                    "disk": {
                        "threshold_enabled" : false
                    }
                }
            }
        }
    }
}
```
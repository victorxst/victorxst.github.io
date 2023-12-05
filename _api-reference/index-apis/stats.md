---
layout: default
title: 统计
parent: 索引API
nav_order: 72
---

# 索引统计
**引入1.0**
{: .label .label-purple }

索引统计信息API提供索引统计信息。对于数据流，API提供了该流的背面索引的统计信息。默认情况下，返回的统计信息是索引级别。接收碎片-等级统计，设置`level` 参数为`shards`。

当碎片移至其他节点时，碎片-清除了碎片的水平统计数据。尽管碎片不再是节点的一部分，但节点保留任何节点-碎片贡献的水平统计数据。
{: .note}

## 路径和HTTP方法

```json
GET /_stats
GET /<index_ids>/_stats
GET /<index_ids>/_stats/<metric>
```

## 路径参数

下表列出了可用路径参数。所有路径参数都是可选的。

| 范围| 数据类型| 描述|
| :--- | :--- | :--- |
| `<index_ids>` | 细绳| 逗号-用于过滤结果的索引，数据流或索引别名的分离列表。支持通配符表达式。默认为`_all` （（`*`）。
`<metric>` | 细绳| 逗号-将包括在响应中的公制组的分开列表。对于有效的值，请参阅[指标](#metrics)。默认为所有指标。|

### 指标

下表列出了所有可用的公制组。

公制| 描述
：--- |：----
`_all` | 返回所有统计数据。
`completion` | 完成建议统计。
`docs` | 返回文档数量和尚未合并的已删除文档数量。索引刷新操作可能会影响此统计数据。
`fielddata` | 现场数据统计。
`flush` | 冲洗统计。
`get` | 获取统计数据，包括缺少统计数据。
`indexing` | 索引统计。
`merge` | 合并统计。
`query_cache` | 查询缓存统计。
`refresh` | 刷新统计。
`request_cache` | 碎片请求缓存统计。
`search` | 搜索统计数据，包括建议操作统计。搜索操作可以与一个或多个组相关联。您可以通过提供一个自定义组的统计信息`groups` 接受逗号的参数-分组名称的列表。要返回所有组的统计信息，请使用`_all`。
`segments` | 有关所有开放段的内存使用的统计信息。如果是`include_segment_file_sizes` 参数为`true`，该指标包括每个Lucene索引文件的汇总磁盘使用情况。
`store` | 字节单元中索引的大小。
`translog` | 翻译统计。
`warmer` | 温暖的统计数据。

## 查询参数

下表列出了可用查询参数。所有查询参数都是可选的。

范围| 数据类型| 描述
:--- | :--- | :--- 
`expand_wildcards` | 细绳| 指定通配符表达式可以扩展到的索引类型。支持逗号-分离的值。有效值是：<br>- `all`：扩展到所有打开和封闭的索引，包括隐藏的索引。<br>- `open`：扩展到打开索引。<br>- `closed`：扩展到封闭索引。<br>- `hidden`：在扩展时包括隐藏的索引。必须与`open`，`closed`， 或两者。<br>- `none`：不要接受通配符的表达。<br>默认值为`open`。
`fields` | 细绳| 逗号-分离的列表或通配符表达式指定统计中包含的字段。如果都不`completion_fields` 也不`fielddata_fields` 提供。
`completion_fields` | 细绳| 逗号-分开的列表或通配符表达式指定字段-等级`completion` 统计数据。
`fielddata_fields` | 细绳| 逗号-分开的列表或通配符表达式指定字段-等级`fielddata` 统计数据。
`forbid_closed_indices` | 布尔| 指定不收集封闭索引的统计信息。默认为`true`。
`groups` | 细绳| 逗号-分开的搜索组列表，包括`search` 统计数据。
`level` | 细绳| 指定用于汇总统计的级别。有效值是：<br>- `cluster`： 簇-水平统计。<br>- `indices`： 指数-水平统计。<br>- `shards`：shard-水平统计。<br>默认值为`indices`。
`include_segment_file_sizes` | 布尔| 指定是否报告每个Lucene索引文件的汇总磁盘使用情况。仅适用于`segments` 统计数据。默认为`false`。
`include_unloaded_segments` | 布尔| 指定是否包含未加载到内存中的段中的信息。默认为`false`。

#### 示例请求：一个索引

```json
GET /testindex/_stats
```
{% include copy-curl.html %}

#### 示例响应

默认情况下，返回的统计信息在`primaries` 和`total` 聚合。这`primaries` 聚合包含主要碎片的统计数据。这`total` 聚合包含主要和复制碎片的统计数据。以下是一个示例索引统计API响应：

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_all": {
    "primaries": {
      "docs": {
        "count": 4,
        "deleted": 0
      },
      "store": {
        "size_in_bytes": 15531,
        "reserved_in_bytes": 0
      },
      "indexing": {
        "index_total": 4,
        "index_time_in_millis": 10,
        "index_current": 0,
        "index_failed": 0,
        "delete_total": 0,
        "delete_time_in_millis": 0,
        "delete_current": 0,
        "noop_update_total": 0,
        "is_throttled": false,
        "throttle_time_in_millis": 0
      },
      "get": {
        "total": 0,
        "time_in_millis": 0,
        "exists_total": 0,
        "exists_time_in_millis": 0,
        "missing_total": 0,
        "missing_time_in_millis": 0,
        "current": 0
      },
      "search": {
        "open_contexts": 0,
        "query_total": 12,
        "query_time_in_millis": 11,
        "query_current": 0,
        "fetch_total": 12,
        "fetch_time_in_millis": 5,
        "fetch_current": 0,
        "scroll_total": 0,
        "scroll_time_in_millis": 0,
        "scroll_current": 0,
        "point_in_time_total": 0,
        "point_in_time_time_in_millis": 0,
        "point_in_time_current": 0,
        "suggest_total": 0,
        "suggest_time_in_millis": 0,
        "suggest_current": 0
      },
      "merges": {
        "current": 0,
        "current_docs": 0,
        "current_size_in_bytes": 0,
        "total": 0,
        "total_time_in_millis": 0,
        "total_docs": 0,
        "total_size_in_bytes": 0,
        "total_stopped_time_in_millis": 0,
        "total_throttled_time_in_millis": 0,
        "total_auto_throttle_in_bytes": 20971520
      },
      "refresh": {
        "total": 8,
        "total_time_in_millis": 58,
        "external_total": 7,
        "external_total_time_in_millis": 60,
        "listeners": 0
      },
      "flush": {
        "total": 1,
        "periodic": 1,
        "total_time_in_millis": 21
      },
      "warmer": {
        "current": 0,
        "total": 6,
        "total_time_in_millis": 0
      },
      "query_cache": {
        "memory_size_in_bytes": 0,
        "total_count": 0,
        "hit_count": 0,
        "miss_count": 0,
        "cache_size": 0,
        "cache_count": 0,
        "evictions": 0
      },
      "fielddata": {
        "memory_size_in_bytes": 0,
        "evictions": 0
      },
      "completion": {
        "size_in_bytes": 0
      },
      "segments": {
        "count": 4,
        "memory_in_bytes": 0,
        "terms_memory_in_bytes": 0,
        "stored_fields_memory_in_bytes": 0,
        "term_vectors_memory_in_bytes": 0,
        "norms_memory_in_bytes": 0,
        "points_memory_in_bytes": 0,
        "doc_values_memory_in_bytes": 0,
        "index_writer_memory_in_bytes": 0,
        "version_map_memory_in_bytes": 0,
        "fixed_bit_set_memory_in_bytes": 0,
        "max_unsafe_auto_id_timestamp": -1,
        "remote_store" : {
          "upload" : {
            "total_upload_size" : {
              "started_bytes" : 152419,
              "succeeded_bytes" : 152419,
              "failed_bytes" : 0
            },
            "refresh_size_lag" : {
              "total_bytes" : 0,
              "max_bytes" : 0
            },
            "max_refresh_time_lag_in_millis" : 0,
            "total_time_spent_in_millis" : 516
          },
          "download" : {
            "total_download_size" : {
              "started_bytes" : 0,
              "succeeded_bytes" : 0,
              "failed_bytes" : 0
            },
            "total_time_spent_in_millis" : 0
          }
        },
        "file_sizes": {}
      },
      "translog": {
        "operations": 0,
        "size_in_bytes": 55,
        "uncommitted_operations": 0,
        "uncommitted_size_in_bytes": 55,
        "earliest_last_modified_age": 142622215,
        "remote_store" : {
          "upload" : {
            "total_uploads" : {
              "started" : 57,
              "failed" : 0,
              "succeeded" : 57
            },
            "total_upload_size" : {
              "started_bytes" : 16830,
              "failed_bytes" : 0,
              "succeeded_bytes" : 16830
            }
          }
        }
      },
      "request_cache": {
        "memory_size_in_bytes": 0,
        "evictions": 0,
        "hit_count": 0,
        "miss_count": 0
      },
      "recovery": {
        "current_as_source": 0,
        "current_as_target": 0,
        "throttle_time_in_millis": 0
      }
    },
    "total": {
      "docs": {
        "count": 4,
        "deleted": 0
      },
      "store": {
        "size_in_bytes": 15531,
        "reserved_in_bytes": 0
      },
      "indexing": {
        "index_total": 4,
        "index_time_in_millis": 10,
        "index_current": 0,
        "index_failed": 0,
        "delete_total": 0,
        "delete_time_in_millis": 0,
        "delete_current": 0,
        "noop_update_total": 0,
        "is_throttled": false,
        "throttle_time_in_millis": 0
      },
      "get": {
        "total": 0,
        "time_in_millis": 0,
        "exists_total": 0,
        "exists_time_in_millis": 0,
        "missing_total": 0,
        "missing_time_in_millis": 0,
        "current": 0
      },
      "search": {
        "open_contexts": 0,
        "query_total": 12,
        "query_time_in_millis": 11,
        "query_current": 0,
        "fetch_total": 12,
        "fetch_time_in_millis": 5,
        "fetch_current": 0,
        "scroll_total": 0,
        "scroll_time_in_millis": 0,
        "scroll_current": 0,
        "point_in_time_total": 0,
        "point_in_time_time_in_millis": 0,
        "point_in_time_current": 0,
        "suggest_total": 0,
        "suggest_time_in_millis": 0,
        "suggest_current": 0
      },
      "merges": {
        "current": 0,
        "current_docs": 0,
        "current_size_in_bytes": 0,
        "total": 0,
        "total_time_in_millis": 0,
        "total_docs": 0,
        "total_size_in_bytes": 0,
        "total_stopped_time_in_millis": 0,
        "total_throttled_time_in_millis": 0,
        "total_auto_throttle_in_bytes": 20971520
      },
      "refresh": {
        "total": 8,
        "total_time_in_millis": 58,
        "external_total": 7,
        "external_total_time_in_millis": 60,
        "listeners": 0
      },
      "flush": {
        "total": 1,
        "periodic": 1,
        "total_time_in_millis": 21
      },
      "warmer": {
        "current": 0,
        "total": 6,
        "total_time_in_millis": 0
      },
      "query_cache": {
        "memory_size_in_bytes": 0,
        "total_count": 0,
        "hit_count": 0,
        "miss_count": 0,
        "cache_size": 0,
        "cache_count": 0,
        "evictions": 0
      },
      "fielddata": {
        "memory_size_in_bytes": 0,
        "evictions": 0
      },
      "completion": {
        "size_in_bytes": 0
      },
      "segments": {
        "count": 4,
        "memory_in_bytes": 0,
        "terms_memory_in_bytes": 0,
        "stored_fields_memory_in_bytes": 0,
        "term_vectors_memory_in_bytes": 0,
        "norms_memory_in_bytes": 0,
        "points_memory_in_bytes": 0,
        "doc_values_memory_in_bytes": 0,
        "index_writer_memory_in_bytes": 0,
        "version_map_memory_in_bytes": 0,
        "fixed_bit_set_memory_in_bytes": 0,
        "max_unsafe_auto_id_timestamp": -1,
        "remote_store" : {
          "upload" : {
            "total_upload_size" : {
              "started_bytes" : 152419,
              "succeeded_bytes" : 152419,
              "failed_bytes" : 0
            },
            "refresh_size_lag" : {
              "total_bytes" : 0,
              "max_bytes" : 0
            },
            "max_refresh_time_lag_in_millis" : 0,
            "total_time_spent_in_millis" : 516
          },
          "download" : {
            "total_download_size" : {
              "started_bytes" : 0,
              "succeeded_bytes" : 0,
              "failed_bytes" : 0
            },
            "total_time_spent_in_millis" : 0
          }
        },
        "file_sizes": {}
      },
      "translog": {
        "operations": 0,
        "size_in_bytes": 55,
        "uncommitted_operations": 0,
        "uncommitted_size_in_bytes": 55,
        "earliest_last_modified_age": 142622215,
        "remote_store" : {
          "upload" : {
            "total_uploads" : {
              "started" : 57,
              "failed" : 0,
              "succeeded" : 57
            },
            "total_upload_size" : {
              "started_bytes" : 16830,
              "failed_bytes" : 0,
              "succeeded_bytes" : 16830
            }
          }
        }
      },
      "request_cache": {
        "memory_size_in_bytes": 0,
        "evictions": 0,
        "hit_count": 0,
        "miss_count": 0
      },
      "recovery": {
        "current_as_source": 0,
        "current_as_target": 0,
        "throttle_time_in_millis": 0
      }
    }
  },
  "indices": {
    "testindex": {
      "uuid": "0SXXSpe9Rp-FpxXXWLOD8Q",
      "primaries": {
        "docs": {
          "count": 4,
          "deleted": 0
        },
        "store": {
          "size_in_bytes": 15531,
          "reserved_in_bytes": 0
        },
        "indexing": {
          "index_total": 4,
          "index_time_in_millis": 10,
          "index_current": 0,
          "index_failed": 0,
          "delete_total": 0,
          "delete_time_in_millis": 0,
          "delete_current": 0,
          "noop_update_total": 0,
          "is_throttled": false,
          "throttle_time_in_millis": 0
        },
        "get": {
          "total": 0,
          "time_in_millis": 0,
          "exists_total": 0,
          "exists_time_in_millis": 0,
          "missing_total": 0,
          "missing_time_in_millis": 0,
          "current": 0
        },
        "search": {
          "open_contexts": 0,
          "query_total": 12,
          "query_time_in_millis": 11,
          "query_current": 0,
          "fetch_total": 12,
          "fetch_time_in_millis": 5,
          "fetch_current": 0,
          "scroll_total": 0,
          "scroll_time_in_millis": 0,
          "scroll_current": 0,
          "point_in_time_total": 0,
          "point_in_time_time_in_millis": 0,
          "point_in_time_current": 0,
          "suggest_total": 0,
          "suggest_time_in_millis": 0,
          "suggest_current": 0
        },
        "merges": {
          "current": 0,
          "current_docs": 0,
          "current_size_in_bytes": 0,
          "total": 0,
          "total_time_in_millis": 0,
          "total_docs": 0,
          "total_size_in_bytes": 0,
          "total_stopped_time_in_millis": 0,
          "total_throttled_time_in_millis": 0,
          "total_auto_throttle_in_bytes": 20971520
        },
        "refresh": {
          "total": 8,
          "total_time_in_millis": 58,
          "external_total": 7,
          "external_total_time_in_millis": 60,
          "listeners": 0
        },
        "flush": {
          "total": 1,
          "periodic": 1,
          "total_time_in_millis": 21
        },
        "warmer": {
          "current": 0,
          "total": 6,
          "total_time_in_millis": 0
        },
        "query_cache": {
          "memory_size_in_bytes": 0,
          "total_count": 0,
          "hit_count": 0,
          "miss_count": 0,
          "cache_size": 0,
          "cache_count": 0,
          "evictions": 0
        },
        "fielddata": {
          "memory_size_in_bytes": 0,
          "evictions": 0
        },
        "completion": {
          "size_in_bytes": 0
        },
        "segments": {
          "count": 4,
          "memory_in_bytes": 0,
          "terms_memory_in_bytes": 0,
          "stored_fields_memory_in_bytes": 0,
          "term_vectors_memory_in_bytes": 0,
          "norms_memory_in_bytes": 0,
          "points_memory_in_bytes": 0,
          "doc_values_memory_in_bytes": 0,
          "index_writer_memory_in_bytes": 0,
          "version_map_memory_in_bytes": 0,
          "fixed_bit_set_memory_in_bytes": 0,
          "max_unsafe_auto_id_timestamp": -1,
          "remote_store" : {
            "upload" : {
              "total_upload_size" : {
                "started_bytes" : 152419,
                "succeeded_bytes" : 152419,
                "failed_bytes" : 0
              },
              "refresh_size_lag" : {
                "total_bytes" : 0,
                "max_bytes" : 0
              },
              "max_refresh_time_lag_in_millis" : 0,
              "total_time_spent_in_millis" : 516
            },
            "download" : {
              "total_download_size" : {
                "started_bytes" : 0,
                "succeeded_bytes" : 0,
                "failed_bytes" : 0
              },
              "total_time_spent_in_millis" : 0
            }
          },
          "file_sizes": {}
        },
        "translog": {
          "operations": 0,
          "size_in_bytes": 55,
          "uncommitted_operations": 0,
          "uncommitted_size_in_bytes": 55,
          "earliest_last_modified_age": 142622215,
          "remote_store" : {
            "upload" : {
              "total_uploads" : {
                "started" : 57,
                "failed" : 0,
                "succeeded" : 57
              },
              "total_upload_size" : {
                "started_bytes" : 16830,
                "failed_bytes" : 0,
                "succeeded_bytes" : 16830
              }
            }
          }
        },
        "request_cache": {
          "memory_size_in_bytes": 0,
          "evictions": 0,
          "hit_count": 0,
          "miss_count": 0
        },
        "recovery": {
          "current_as_source": 0,
          "current_as_target": 0,
          "throttle_time_in_millis": 0
        }
      },
      "total": {
        "docs": {
          "count": 4,
          "deleted": 0
        },
        "store": {
          "size_in_bytes": 15531,
          "reserved_in_bytes": 0
        },
        "indexing": {
          "index_total": 4,
          "index_time_in_millis": 10,
          "index_current": 0,
          "index_failed": 0,
          "delete_total": 0,
          "delete_time_in_millis": 0,
          "delete_current": 0,
          "noop_update_total": 0,
          "is_throttled": false,
          "throttle_time_in_millis": 0
        },
        "get": {
          "total": 0,
          "time_in_millis": 0,
          "exists_total": 0,
          "exists_time_in_millis": 0,
          "missing_total": 0,
          "missing_time_in_millis": 0,
          "current": 0
        },
        "search": {
          "open_contexts": 0,
          "query_total": 12,
          "query_time_in_millis": 11,
          "query_current": 0,
          "fetch_total": 12,
          "fetch_time_in_millis": 5,
          "fetch_current": 0,
          "scroll_total": 0,
          "scroll_time_in_millis": 0,
          "scroll_current": 0,
          "point_in_time_total": 0,
          "point_in_time_time_in_millis": 0,
          "point_in_time_current": 0,
          "suggest_total": 0,
          "suggest_time_in_millis": 0,
          "suggest_current": 0
        },
        "merges": {
          "current": 0,
          "current_docs": 0,
          "current_size_in_bytes": 0,
          "total": 0,
          "total_time_in_millis": 0,
          "total_docs": 0,
          "total_size_in_bytes": 0,
          "total_stopped_time_in_millis": 0,
          "total_throttled_time_in_millis": 0,
          "total_auto_throttle_in_bytes": 20971520
        },
        "refresh": {
          "total": 8,
          "total_time_in_millis": 58,
          "external_total": 7,
          "external_total_time_in_millis": 60,
          "listeners": 0
        },
        "flush": {
          "total": 1,
          "periodic": 1,
          "total_time_in_millis": 21
        },
        "warmer": {
          "current": 0,
          "total": 6,
          "total_time_in_millis": 0
        },
        "query_cache": {
          "memory_size_in_bytes": 0,
          "total_count": 0,
          "hit_count": 0,
          "miss_count": 0,
          "cache_size": 0,
          "cache_count": 0,
          "evictions": 0
        },
        "fielddata": {
          "memory_size_in_bytes": 0,
          "evictions": 0
        },
        "completion": {
          "size_in_bytes": 0
        },
        "segments": {
          "count": 4,
          "memory_in_bytes": 0,
          "terms_memory_in_bytes": 0,
          "stored_fields_memory_in_bytes": 0,
          "term_vectors_memory_in_bytes": 0,
          "norms_memory_in_bytes": 0,
          "points_memory_in_bytes": 0,
          "doc_values_memory_in_bytes": 0,
          "index_writer_memory_in_bytes": 0,
          "version_map_memory_in_bytes": 0,
          "fixed_bit_set_memory_in_bytes": 0,
          "max_unsafe_auto_id_timestamp": -1,
          "remote_store" : {
            "upload" : {
              "total_upload_size" : {
                "started_bytes" : 152419,
                "succeeded_bytes" : 152419,
                "failed_bytes" : 0
              },
              "refresh_size_lag" : {
                "total_bytes" : 0,
                "max_bytes" : 0
              },
              "max_refresh_time_lag_in_millis" : 0,
              "total_time_spent_in_millis" : 516
            },
            "download" : {
              "total_download_size" : {
                "started_bytes" : 0,
                "succeeded_bytes" : 0,
                "failed_bytes" : 0
              },
              "total_time_spent_in_millis" : 0
            }
          },
          "file_sizes": {}
        },
        "translog": {
          "operations": 0,
          "size_in_bytes": 55,
          "uncommitted_operations": 0,
          "uncommitted_size_in_bytes": 55,
          "earliest_last_modified_age": 142622215,
          "remote_store" : {
            "upload" : {
              "total_uploads" : {
                "started" : 57,
                "failed" : 0,
                "succeeded" : 57
              },
              "total_upload_size" : {
                "started_bytes" : 16830,
                "failed_bytes" : 0,
                "succeeded_bytes" : 16830
              }
            }
          }
        },
        "request_cache": {
          "memory_size_in_bytes": 0,
          "evictions": 0,
          "hit_count": 0,
          "miss_count": 0
        },
        "recovery": {
          "current_as_source": 0,
          "current_as_target": 0,
          "throttle_time_in_millis": 0
        }
      }
    }
  }
}
```
</details>

#### 示例请求：逗号-分开索引列表

```json
GET /testindex1,testindex2/_stats
```
{% include copy-curl.html %}

#### 示例请求：通配符表达式

```json
GET /testindex*/_stats
```
{% include copy-curl.html %}

#### 示例请求：特定属性

```json
GET /testindex/_stats/refresh,flush
```
{% include copy-curl.html %}

#### 示例请求：扩展通配符

```json
GET /testindex*/_stats?expand_wildcards=open,hidden
```
{% include copy-curl.html %}

#### 示例请求：碎片-水平统计

```json
GET /testindex/_stats?level=shards
```
{% include copy-curl.html %}

## 并发段搜索

从OpenSearch 2.10开始[并发段搜索]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search/) 允许每个碎片-在查询阶段并行搜索段的级别请求。如果你[启用实验并发段搜索功能标志]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search#enabling-the-feature-flag)，索引统计API响应将包含一些其他字段，其中包含有关切片的统计信息（线程执行的工作单位）。无论是否启用了用于并发段搜索的群集和索引设置，都将提供这些字段。有关切片的更多信息，请参阅[并发段搜索]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search#searching-segments-concurrently)。

下表提供了有关添加响应字段的信息。

|响应字段| 描述|
|:---	|:---	| 
|`search.concurrent_avg_slice_count`|所有搜索请求的平均切片计数。这是根据总切片计数除以并发搜索请求的总数。|
|`search.concurrent_query_total`|使用并发段搜索的查询操作总数。|
|`search.concurrent_query_time_in_millis`|所有查询操作使用并发段搜索的所有查询操作所需的总时间，以毫秒为单位。|
|`search.concurrent_query_current`|使用并发段搜索的当前运行查询操作的数量。|


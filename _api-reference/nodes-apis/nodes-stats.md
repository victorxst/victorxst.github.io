---
layout: default
title: 节点统计
parent: 节点API
nav_order: 20
---

# 节点统计
**引入1.0**
{: .label .label-purple }

节点统计API返回有关您的群集的统计信息。

## 路径和HTTP方法

```json
GET /_nodes/stats
GET /_nodes/<node_id>/stats
GET /_nodes/stats/<metric>
GET /_nodes/<node_id>/stats/<metric>
GET /_nodes/stats/<metric>/<index_metric>
GET /_nodes/<node_id>/stats/<metric>/<index_metric>
```

## 路径参数

下表列出了可用路径参数。所有路径参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
nodeid| 细绳| 逗号-用于过滤结果的节点的分离列表。支持[节点过滤器]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/#node-filters)。默认为`_all`。
公制| 细绳| 逗号-响应中包含的度量组的分开列表。例如，`jvm,fs`。请参阅以下所有索引指标的列表。默认为所有指标。
index_metric| 细绳| 逗号-响应中包含的索引度量组的分离列表。例如，`docs,store`。请参阅以下所有索引指标的列表。默认为所有索引指标。

下表列出了所有可用的公制组。

公制| 描述
:--- |:----
指数| 索引统计信息，例如大小，文档计数和搜索，索引和删除文档的时间。
操作系统| 有关主机OS的统计信息，包括负载，内存和交换。
过程| 有关过程的统计信息，包括其内存消耗，打开文件描述符和CPU使用。
JVM| 有关JVM的统计信息，包括内存池，缓冲池和垃圾收集以及加载类的数量。
FS| 文件系统统计信息，例如读取/写入统计信息，数据路径和免费磁盘空间。
运输| 有关群集通信中发送/接收的传输层统计信息。
http| 有关HTTP层的统计信息。
断路器| 有关现场数据断路器的统计数据。
脚本| 有关脚本的统计信息，例如编译和缓存驱逐。
发现| 有关集群状态的统计数据。
摄取| 有关摄入管道的统计数据。
Adaptive_selection| 有关自适应副本选择的统计信息，该统计数据使用碎片分配意识选择合格的节点。
script_cache| 有关脚本缓存的统计信息。
indexing_pressure| 有关节点索引压力的统计数据。
shard_indexing_pressure| 有关碎片索引压力的统计数据。

过滤返回的信息`indices` 公制，您可以使用特定`index_metric` 值。仅当您使用以下查询类型时，才可以使用这些：

```json
GET _nodes/stats/
GET _nodes/stats/_all
GET _nodes/stats/indices
```

支持以下索引指标：

- 文档
- 店铺
- 索引
- 得到
- 搜索
- 合并
- 刷新
- 冲洗
- 温暖
- query_cache
- fieldData
- 完成
- 细分市场
- 翻译
- request_cache

例如，以下查询请求统计信息`docs` 和`search`：

```json
GET _nodes/stats/indices/docs,search
```
{% include copy-curl.html %}

## 查询参数

下表列出了可用查询参数。所有查询参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
postion_fields| 细绳| 包括在完成统计中的字段。支持逗号-分开的列表和通配符表达式。
fielddata_fields| 细绳| 将包括在fieldData统计中的字段。支持逗号-分开的列表和通配符表达式。
字段| 细绳| 包括的字段。支持逗号-分开的列表和通配符表达式。
组| 细绳| 逗号-分开的搜索组列表，其中包括在搜索统计中。
等级| 细绳| 指定统计信息是在集群，索引还是碎片级别汇总的。有效值是`indices`，`node`， 和`shard`。
暂停| 时间| 设置节点响应的时间限制。默认为`30s`。
include_segment_file_sizes| 布尔| 如果要求段统计信息，则此字段指定返回每个Lucene索引文件的汇总磁盘使用情况。默认为`false`。

#### 示例请求

```json
GET _nodes/stats/
```
{% include copy-curl.html %}

#### 示例响应

选择箭头以查看示例响应。

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "docker-cluster",
  "nodes" : {
    "F-ByTQzVQ3GQeYzQJArJGQ" : {
      "timestamp" : 1664484195257,
      "name" : "opensearch",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "roles" : [
        "cluster_manager",
        "data",
        "ingest",
        "remote_cluster_client"
      ],
      "attributes" : {
        "shard_indexing_pressure_enabled" : "true"
      },
      "indices" : {
        "docs" : {
          "count" : 13160,
          "deleted" : 12
        },
        "store" : {
          "size_in_bytes" : 6263461,
          "reserved_in_bytes" : 0
        },
        "indexing" : {
          "index_total" : 0,
          "index_time_in_millis" : 0,
          "index_current" : 0,
          "index_failed" : 0,
          "delete_total" : 204,
          "delete_time_in_millis" : 427,
          "delete_current" : 0,
          "noop_update_total" : 0,
          "is_throttled" : false,
          "throttle_time_in_millis" : 0
        },
        "get" : {
          "total" : 4,
          "time_in_millis" : 18,
          "exists_total" : 4,
          "exists_time_in_millis" : 18,
          "missing_total" : 0,
          "missing_time_in_millis" : 0,
          "current" : 0
        },
        "search" : {
          "open_contexts": 4,
          "query_total": 194,
          "query_time_in_millis": 467,
          "query_current": 0,
          "fetch_total": 194,
          "fetch_time_in_millis": 143,
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
        "merges" : {
          "current" : 0,
          "current_docs" : 0,
          "current_size_in_bytes" : 0,
          "total" : 1,
          "total_time_in_millis" : 5,
          "total_docs" : 12,
          "total_size_in_bytes" : 3967,
          "total_stopped_time_in_millis" : 0,
          "total_throttled_time_in_millis" : 0,
          "total_auto_throttle_in_bytes" : 251658240
        },
        "refresh" : {
          "total" : 74,
          "total_time_in_millis" : 201,
          "external_total" : 57,
          "external_total_time_in_millis" : 314,
          "listeners" : 0
        },
        "flush" : {
          "total" : 28,
          "periodic" : 28,
          "total_time_in_millis" : 1261
        },
        "warmer" : {
          "current" : 0,
          "total" : 45,
          "total_time_in_millis" : 99
        },
        "query_cache" : {
          "memory_size_in_bytes" : 0,
          "total_count" : 0,
          "hit_count" : 0,
          "miss_count" : 0,
          "cache_size" : 0,
          "cache_count" : 0,
          "evictions" : 0
        },
        "fielddata" : {
          "memory_size_in_bytes" : 356,
          "evictions" : 0
        },
        "completion" : {
          "size_in_bytes" : 0,
          "fields" : { }
        },
        "segments" : {
          "count" : 17,
          "memory_in_bytes" : 0,
          "terms_memory_in_bytes" : 0,
          "stored_fields_memory_in_bytes" : 0,
          "term_vectors_memory_in_bytes" : 0,
          "norms_memory_in_bytes" : 0,
          "points_memory_in_bytes" : 0,
          "doc_values_memory_in_bytes" : 0,
          "index_writer_memory_in_bytes" : 0,
          "version_map_memory_in_bytes" : 0,
          "fixed_bit_set_memory_in_bytes" : 288,
          "max_unsafe_auto_id_timestamp" : -1,
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
          "file_sizes" : { }
        },
        "translog" : {
          "operations" : 12,
          "size_in_bytes" : 1452,
          "uncommitted_operations" : 12,
          "uncommitted_size_in_bytes" : 1452,
          "earliest_last_modified_age" : 164160,
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
        "request_cache" : {
          "memory_size_in_bytes" : 1649,
          "evictions" : 0,
          "hit_count" : 0,
          "miss_count" : 18
        },
        "recovery" : {
          "current_as_source" : 0,
          "current_as_target" : 0,
          "throttle_time_in_millis" : 0
        }
      },
      "os" : {
        "timestamp" : 1664484195263,
        "cpu" : {
          "percent" : 0,
          "load_average" : {
            "1m" : 0.0,
            "5m" : 0.0,
            "15m" : 0.0
          }
        },
        "mem" : {
          "total_in_bytes" : 13137076224,
          "free_in_bytes" : 9265442816,
          "used_in_bytes" : 3871633408,
          "free_percent" : 71,
          "used_percent" : 29
        },
        "swap" : {
          "total_in_bytes" : 4294967296,
          "free_in_bytes" : 4294967296,
          "used_in_bytes" : 0
        },
        "cgroup" : {
          "cpuacct" : {
            "control_group" : "/",
            "usage_nanos" : 338710071600
          },
          "cpu" : {
            "control_group" : "/",
            "cfs_period_micros" : 100000,
            "cfs_quota_micros" : -1,
            "stat" : {
              "number_of_elapsed_periods" : 0,
              "number_of_times_throttled" : 0,
              "time_throttled_nanos" : 0
            }
          },
          "memory" : {
            "control_group" : "/",
            "limit_in_bytes" : "9223372036854771712",
            "usage_in_bytes" : "1432346624"
          }
        }
      },
      "process" : {
        "timestamp" : 1664484195263,
        "open_file_descriptors" : 556,
        "max_file_descriptors" : 65536,
        "cpu" : {
          "percent" : 0,
          "total_in_millis" : 170870
        },
        "mem" : {
          "total_virtual_in_bytes" : 6563344384
        }
      },
      "jvm" : {
        "timestamp" : 1664484195264,
        "uptime_in_millis" : 21232111,
        "mem" : {
          "heap_used_in_bytes" : 308650480,
          "heap_used_percent" : 57,
          "heap_committed_in_bytes" : 536870912,
          "heap_max_in_bytes" : 536870912,
          "non_heap_used_in_bytes" : 147657128,
          "non_heap_committed_in_bytes" : 152502272,
          "pools" : {
            "young" : {
              "used_in_bytes" : 223346688,
              "max_in_bytes" : 0,
              "peak_used_in_bytes" : 318767104,
              "peak_max_in_bytes" : 0,
              "last_gc_stats" : {
                "used_in_bytes" : 0,
                "max_in_bytes" : 0,
                "usage_percent" : -1
              }
            },
            "old" : {
              "used_in_bytes" : 67068928,
              "max_in_bytes" : 536870912,
              "peak_used_in_bytes" : 67068928,
              "peak_max_in_bytes" : 536870912,
              "last_gc_stats" : {
                "used_in_bytes" : 34655744,
                "max_in_bytes" : 536870912,
                "usage_percent" : 6
              }
            },
            "survivor" : {
              "used_in_bytes" : 18234864,
              "max_in_bytes" : 0,
              "peak_used_in_bytes" : 32721280,
              "peak_max_in_bytes" : 0,
              "last_gc_stats" : {
                "used_in_bytes" : 18234864,
                "max_in_bytes" : 0,
                "usage_percent" : -1
              }
            }
          }
        },
        "threads" : {
          "count" : 80,
          "peak_count" : 80
        },
        "gc" : {
          "collectors" : {
            "young" : {
              "collection_count" : 18,
              "collection_time_in_millis" : 199
            },
            "old" : {
              "collection_count" : 0,
              "collection_time_in_millis" : 0
            }
          }
        },
        "buffer_pools" : {
          "mapped" : {
            "count" : 23,
            "used_in_bytes" : 6232113,
            "total_capacity_in_bytes" : 6232113
          },
          "direct" : {
            "count" : 63,
            "used_in_bytes" : 9050069,
            "total_capacity_in_bytes" : 9050068
          },
          "mapped - 'non-volatile memory'" : {
            "count" : 0,
            "used_in_bytes" : 0,
            "total_capacity_in_bytes" : 0
          }
        },
        "classes" : {
          "current_loaded_count" : 20693,
          "total_loaded_count" : 20693,
          "total_unloaded_count" : 0
        }
      },
      "thread_pool" : {
        "OPENSEARCH_ML_TASK_THREAD_POOL" : {
          "threads" : 0,
          "queue" : 0,
          "active" : 0,
          "rejected" : 0,
          "largest" : 0,
          "completed" : 0
        },
        "ad-batch-task-threadpool" : {
          "threads" : 0,
          "queue" : 0,
          "active" : 0,
          "rejected" : 0,
          "largest" : 0,
          "completed" : 0
        },
        ...
      },
      "fs" : {
        "timestamp" : 1664484195264,
        "total" : {
          "total_in_bytes" : 269490393088,
          "free_in_bytes" : 261251477504,
          "available_in_bytes" : 247490805760
        },
        "data" : [
          {
            "path" : "/usr/share/opensearch/data/nodes/0",
            "mount" : "/ (overlay)",
            "type" : "overlay",
            "total_in_bytes" : 269490393088,
            "free_in_bytes" : 261251477504,
            "available_in_bytes" : 247490805760
          }
        ],
        "io_stats" : { }
      },
      "transport" : {
        "server_open" : 0,
        "total_outbound_connections" : 0,
        "rx_count" : 0,
        "rx_size_in_bytes" : 0,
        "tx_count" : 0,
        "tx_size_in_bytes" : 0
      },
      "http" : {
        "current_open" : 5,
        "total_opened" : 1108
      },
      "breakers" : {
        "request" : {
          "limit_size_in_bytes" : 322122547,
          "limit_size" : "307.1mb",
          "estimated_size_in_bytes" : 0,
          "estimated_size" : "0b",
          "overhead" : 1.0,
          "tripped" : 0
        },
        "fielddata" : {
          "limit_size_in_bytes" : 214748364,
          "limit_size" : "204.7mb",
          "estimated_size_in_bytes" : 356,
          "estimated_size" : "356b",
          "overhead" : 1.03,
          "tripped" : 0
        },
        "in_flight_requests" : {
          "limit_size_in_bytes" : 536870912,
          "limit_size" : "512mb",
          "estimated_size_in_bytes" : 0,
          "estimated_size" : "0b",
          "overhead" : 2.0,
          "tripped" : 0
        },
        "parent" : {
          "limit_size_in_bytes" : 510027366,
          "limit_size" : "486.3mb",
          "estimated_size_in_bytes" : 308650480,
          "estimated_size" : "294.3mb",
          "overhead" : 1.0,
          "tripped" : 0
        }
      },
      "script" : {
        "compilations" : 0,
        "cache_evictions" : 0,
        "compilation_limit_triggered" : 0
      },
      "discovery" : {
        "cluster_state_queue" : {
          "total" : 0,
          "pending" : 0,
          "committed" : 0
        },
        "published_cluster_states" : {
          "full_states" : 2,
          "incompatible_diffs" : 0,
          "compatible_diffs" : 10
        }
      },
      "ingest" : {
        "total" : {
          "count" : 0,
          "time_in_millis" : 0,
          "current" : 0,
          "failed" : 0
        },
        "pipelines" : { }
      },
      "search_pipeline" : {
        "total_request" : {
          "count" : 5,
          "time_in_millis" : 158,
          "current" : 0,
          "failed" : 0
        },
        "total_response" : {
          "count" : 2,
          "time_in_millis" : 1,
          "current" : 0,
          "failed" : 0
        },
        "pipelines" : {
          "public_info" : {
            "request" : {
              "count" : 3,
              "time_in_millis" : 71,
              "current" : 0,
              "failed" : 0
            },
            "response" : {
              "count" : 0,
              "time_in_millis" : 0,
              "current" : 0,
              "failed" : 0
            },
            "request_processors" : [
              {
                "filter_query:abc" : {
                  "type" : "filter_query",
                  "stats" : {
                    "count" : 1,
                    "time_in_millis" : 0,
                    "current" : 0,
                    "failed" : 0
                  }
                }
              },
            ]
              ...
            "response_processors" : [
              {
                "rename_field" : {
                  "type" : "rename_field",
                  "stats" : {
                    "count" : 2,
                    "time_in_millis" : 1,
                    "current" : 0,
                    "failed" : 0
                  }
                }
              }
            ]
          },
          ...
        }
      },
      "adaptive_selection" : {
        "F-ByTQzVQ3GQeYzQJArJGQ" : {
          "outgoing_searches" : 0,
          "avg_queue_size" : 0,
          "avg_service_time_ns" : 501024,
          "avg_response_time_ns" : 794105,
          "rank" : "0.8"
        }
      },
      "script_cache" : {
        "sum" : {
          "compilations" : 0,
          "cache_evictions" : 0,
          "compilation_limit_triggered" : 0
        },
        "contexts" : [
          {
            "context" : "aggregation_selector",
            "compilations" : 0,
            "cache_evictions" : 0,
            "compilation_limit_triggered" : 0
          },
          {
            "context" : "aggs",
            "compilations" : 0,
            "cache_evictions" : 0,
            "compilation_limit_triggered" : 0
          },
          ...
        ]
      },
      "indexing_pressure" : {
        "memory" : {
          "current" : {
            "combined_coordinating_and_primary_in_bytes" : 0,
            "coordinating_in_bytes" : 0,
            "primary_in_bytes" : 0,
            "replica_in_bytes" : 0,
            "all_in_bytes" : 0
          },
          "total" : {
            "combined_coordinating_and_primary_in_bytes" : 40256,
            "coordinating_in_bytes" : 40256,
            "primary_in_bytes" : 45016,
            "replica_in_bytes" : 0,
            "all_in_bytes" : 40256,
            "coordinating_rejections" : 0,
            "primary_rejections" : 0,
            "replica_rejections" : 0
          },
          "limit_in_bytes" : 53687091
        }
      },
      "shard_indexing_pressure" : {
        "stats" : { },
        "total_rejections_breakup_shadow_mode" : {
          "node_limits" : 0,
          "no_successful_request_limits" : 0,
          "throughput_degradation_limits" : 0
        },
        "enabled" : false,
        "enforced" : false
      }
    }
  }
}
```
</delect>

## 响应字段

下表列出了所有响应字段。

| 场地| 数据类型| 描述|
| :--- | :--- | :--- |
| _ nodes| 目的| 有关返回节点的统计信息。|
| _nodes.total| 整数| 此请求的节点总数。|
| _nodes.uccessful| 整数| 请求成功的节点数量。|
| _nodes.failed| 整数| 请求失败的节点数量。如果请求失败的节点，则包含故障消息。|
| cluster_name| 细绳| 集群的名称。|
| [节点](#nodes) | 目的| 此请求中包含的节点的统计信息。|

### `nodes`

这`nodes` 对象包含请求返回的所有节点及其ID。每个节点具有以下属性。

场地| 数据类型| 描述
:--- | :--- | :---
时间戳| 整数| 自时代以来，收集节点统计的时间是在毫秒内。
姓名| 细绳| 节点的名称。
transport_address| IP地址| 节点在集群中使用的传输层的主机和端口进行内部通信。
主持人| IP地址| 节点的网络主机。
IP| IP地址| 节点的IP地址和端口。
角色| 大批| 节点的角色（例如，`cluster_manager`，`data`， 或者`ingest`）。
属性| 目的| 节点的属性（例如，`shard_indexing_pressure_enabled`）。
[指数](#indices) | 目的| 节点上有碎片的每个索引的索引统计信息。
[操作系统](#os) | 目的| 有关该节点的OS的统计信息。
[过程](#process) | 目的| 该节点的过程统计信息。
[JVM](#jvm) | 目的| 有关节点的JVM的统计信息。
[thread_pool](#thread_pool)| 目的| 关于节点的每个线程池的统计信息。
[FS](#fs) | 目的| 有关该节点的文件存储的统计信息。
[运输](#transport) | 目的| 节点的运输统计数据。
http| 目的| 节点的HTTP统计信息。
http.current_open| 整数| 该节点当前打开的HTTP连接的数量。
http.total_opened| 整数| 自节点启动以来，HTTP连接的总数已打开。
[断路器](#breakers) | 目的| 有关节点断路器的统计信息。
[脚本](#script-and-script_cache)| 目的| 节点的脚本统计信息。
[script_cache](#script-and-script_cache)| 目的| 节点的脚本缓存统计信息。
[发现](#discovery) | 目的| 节点的节点发现统计信息。
[摄取](#ingest) | 目的| 节点的摄入统计信息。
[search_pipeline](#search_pipeline) | 目的| 统计数据与[搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/)。
[Adaptive_selection](#adaptive_selection) | 目的| 有关节点自适应选择的统计信息。
[indexing_pressure](#indexing_pressure) | 目的| 与节点的索引压力有关的统计数据。
[shard_indexing_pressure](#shard_indexing_pressure) | 目的| 统计数据与碎片级别的索引压力有关。
[search_backpressure]({{site.url}}{{site.baseurl}}/opensearch/search-backpressure#search-backpressure-stats-api) | 目的| 与搜索背压有关的统计信息。

### `indices`

这`indices` 对象包含每个索引的索引统计信息，并在此节点上带有碎片。每个索引具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
文档| 目的| 节点上存在的所有主要碎片的文档统计信息。
文档| 整数| Lucene报告的文件数量。不包括尚未分配给细分市场的已删除文档和最近的索引文档。嵌套文档分别计数。
文档| 整数| Lucene报告的已删除文件数量。不包括尚未影响该细分市场的最新删除操作。
店铺| 目的| 关于节点上碎片的碎片尺寸的统计数据。
store.size_in_bytes| 整数| 节点上所有碎片的总尺寸。
store.serveve_in_bytes| 整数| 预测的字节数将增加，因为诸如恢复快照和同伴恢复之类的活动。
索引| 目的| 有关该节点的索引操作的统计信息。
索引。index_total| 整数| 节点上的索引操作总数。
索引.index_time_in_millis| 整数| 所有索引操作的总时间，以毫秒为单位。
索引。index_current| 整数| 当前正在运行的索引操作的数量。
索引。index_failed| 整数| 失败的索引操作数量。
索引。delete_total| 整数| 删除总数。
indexing.delete_time_in_millis| 整数| 所有删除操作的总时间，以毫秒为单位。
索引。delete_current| 整数| 当前正在运行的删除操作的数量。
索引.noop_update_total| 整数| NOOP操作的总数。
索引。IS_THROTTLED| 布尔| 指定是否有任何操作。
索引。Throttle_time_in_millis| 整数| 节流操作的总时间，以毫秒为单位。
得到| 目的| 有关该节点的获取操作的统计信息。
get.Total| 整数| 获得操作的总数。
get.time_in_millis| 整数| 所有获得操作的总时间，以毫秒为单位。
get.exists_total| 整数| 成功获得操作的总数。
get.exists_time_in_millis| 整数| 全部成功进行操作的总时间，以毫秒为单位。
get.missing_total| 整数| 失败的获取操作数量。
get.missing_time_in_millis| 整数| 所有失败的获取操作的总时间，以毫秒为单位。
get.current| 整数| 当前正在运行的获取操作数量。
搜索| 目的| 有关该节点的搜索操作的统计信息。
search.point_in_time_total| 整数| 自节点上次重新启动以来已经创建（已完成和活动）的时间点上下文的总数。
search.point_in_time_time_in_in_millis| 整数|  自节点上次重新启动以来，该时间点上下文的时间上下文的时间已被打开。
search.point_in_time_current| 整数| 当前打开的时间点上下文的数量。
search.open_contexts| 整数| 开放搜索上下文的数量。
search.query_total| 整数| 查询操作的总数。
search.query_time_in_millis| 整数| 所有查询操作的总时间，以毫秒为单位。
search.query_current| 整数| 当前正在运行的查询操作数量。
search.fetch_total| 整数| 获取操作的总数。
search.fetch_time_in_millis| 整数| 所有提取操作的总时间，以毫秒为单位。
search.fetch_current| 整数| 当前正在运行的获取操作数量。
search.scroll_total| 整数| 滚动操作的总数。
search.scroll_time_in_millis| 整数| 所有滚动操作的总时间，以毫秒为单位。
search.scroll_current| 整数| 当前正在运行的滚动操作数量。
search.suggest_total| 整数| 建议操作的总数。
search.suggest_time_in_millis| 整数| 所有建议操作的总时间，以毫秒为单位。
search.suggest_current| 整数| 当前正在运行的建议操作数量。
合并| 目的| 有关该节点合并操作的统计信息。
合并| 整数| 当前正在运行的合并操作数量。
Merges.Current_Docs| 整数| 目前正在运行的文档合并数量。
Merges.current_size_in_bytes| 整数| 内存大小在字节中，用于执行当前合并操作。
合并| 整数| 合并操作的总数。
Merges.total_time_in_millis| 整数| 合并的总时间，以毫秒为单位。
Merges.Total_Docs| 整数| 已合并的文件总数。
Merges.total_size_in_bytes| 整数| 所有合并文档的总大小，字节。
Merges.total_stopped_time_in_millis| 整数| 以毫秒为单位停止合并操作的总时间。
Merges.total_throttled_time_in_millis| 整数| 以毫秒为单位的节流合并操作花费的总时间。
Merges.total_auto_throttle_in_bytes| 整数| 字节以自动进行合并操作的总大小。
刷新| 目的| 有关该节点的刷新操作的统计信息。
刷新| 整数| 刷新操作的总数。
refresh.total_time_in_millis| 整数| 所有刷新操作的总时间，以毫秒为单位。
refresh.external_total| 整数| 外部刷新操作的总数。
refresh.external_total_time_in_in_millis| 整数| 所有外部刷新操作的总时间，以毫秒为单位。
刷新| 整数| 刷新听众的数量。
冲洗| 目的| 有关节点的冲洗操作的统计信息。
齐平| 整数| 冲洗操作的总数。
冲洗| 整数| 周期性冲洗操作的总数。
flush.total_time_in_millis| 整数| 所有冲洗操作的总时间，以毫秒为单位。
温暖| 目的| 有关该节点的索引变暖操作的统计数据。
温暖| 整数| 当前指数变暖操作的数量。
温暖| 整数| 索引变暖操作的总数。
harmer.total_time_in_millis| 整数| 所有指数变暖操作的总时间，毫秒。
query_cache| 有关该节点的查询缓存操作的统计信息。
query_cache.memory_size_in_bytes| 整数| 该节点中所有碎片的查询缓存的内存量。
query_cache.total_count| 整数| 查询缓存中的命中，错过和缓存查询的总数。
query_cache.hit_count| 整数| 查询缓存中的命中总数。
query_cache.miss_count| 整数| 查询缓存中错过的总数。
query_cache.cache_size| 整数| 查询缓存的大小，字节。
query_cache.cache_count| 整数| 查询缓存中的查询数。
query_cache.evictions| 整数| 查询缓存中的驱逐数量。
fieldData| 目的| 有关节点中所有碎片的字段数据缓存的统计信息。
fielddata.memory_size_in_bytes| 整数| 节点中所有碎片的字段数据缓存的内存总量。
fieldData.Evictions| 整数| 字段数据缓存中的驱逐数量。
fielddata.fields| 目的| 包含所有字段数据字段。
完成| 目的| 关于节点中所有碎片的完成的统计信息。
plote.size_in_bytes| 整数| 在节点中的所有碎片，字节中用于完成的内存总量。
完成| 目的| 包含完成字段。
细分市场| 目的| 关于节点中所有碎片的段的统计数据。
段| 整数| 段总数。
segments.memory_in_bytes| 整数| 内存总量，字节。
segments.terms_memory_in_bytes| 整数| 字节中用于术语的内存总量。
segments.stored_fields_memory_in_bytes| 整数| 在字节中用于存储字段的内存总量。
segments.term_vectors_memory_in_bytes| 整数| 字节中用于术语向量的内存总量。
segments.norms_memory_in_bytes| 整数| 字节中用于标准化因子的内存总量。
sevments.points_memory_in_bytes| 整数| 字节中用于点的内存总数。
segments.doc_values_memory_in_bytes| 整数| 字节中用于DOC值的内存总量。
schments.index_writer_memory_in_bytes| 整数| 所有索引作者使用的内存总数，字节。
segments.version_map_memory_in_bytes| 整数| 所有版本地图所使用的内存总量，字节中。
segments.fixed_bit_set_memory_in_bytes| 整数| 固定位集使用的内存总量，字节中。固定位集用于嵌套对象并加入字段。
segments.max_unsafe_auto_id_id_timestamp| 整数| 自时代以来，最新退休索引请求的时间戳以毫秒为单位。
segments.segment_replication| 目的| 在节点上启用片段复制时，所有主要碎片的段复制统计信息。
segments.sement_replication.maxbytesbehind| 长的| 主复制品后面的最大字节数。
segments.sement_replication.totalbytesbehind| 长的| 主复制品背后的字节总数。
segments.sement_replication.maxreplicationlag| 长的| 由副本以毫秒为单位的最长时间，以赶上其主要时间。
segments.remote_store| 目的| 有关远程细分商店操作的统计信息。
segments.remote_store.upload| 目的| 统计信息与上传到远程细分商店有关。
segments.remote_store.upload.total_upload_size| 目的| 在字节中的数据量上传到远程段存储。
segments.remote_store.upload.total_upload_size.started_bytes| 整数| 上传启动后，要上传到远程段存储的字节数。
segments.remote_store.upload.total_upload_size.succeeded_bytes| 整数| 字节数成功地上传到了远程段存储。
segments.remote_store.upload.total_upload_size.failed_bytes| 整数| 未能上传到远程段存储的字节数。
segments.remote_store.upload.refresh_size_lag| 目的| 在远程细分商店和本地商店之间上传期间的滞后量。
segments.remote_store.upload.refresh_size_lag.total_bytes| 整数| 在远程段商店和本地商店之间上传刷新期间滞后的字节总数。
segments.remote_store.upload.refresh_size_lag.max_bytes| 整数| 在远程细分商店和本地商店之间的上传刷新期间，在字节中的最大滞后量。
segments.remote_store.upload.max_refresh_time_lag_in_millis| 整数| 远程刷新在本地刷新后面的最大持续时间，毫秒毫秒。
segments.remote_store.upload.total_time_spent_in_millis| 整数| 以毫秒为单位的总时间用于上传到远程细分商店。
segments.remote_store.download| 目的| 与远程细分商店下载有关的统计信息。
segments.remote_store.download.total_download_size| 目的| 从远程段存储中下载的数据总量。
segments.remote_store.download.total_download_size.started_bytes| 整数| 下载启动后，从远程段商店下载的字节数。
segments.remote_store.download.total_download_size.succeeded_bytes| 整数| 从远程段商店成功下载的字节数。
segments.remote_store.download.total_download_size.failed_bytes| 整数| 未从远程段存储下载的字节数。
segments.remote_store.download.total_time_spent_in_millis| 整数| 总持续时间以毫秒为单位，用于从远程段商店的下载量。
segments.file_sizes| 整数| 有关细分文件大小的统计信息。
翻译| 目的| 有关该节点的事务日志操作的统计信息。
translog.serations| 整数| 翻译操作的数量。
Translog.size_in_bytes| 整数| 翻译的大小，字节。
transog.uncommitty_operations| 整数| 未投入的翻译操作的数量。
transog.uncommitty_size_in_bytes| 整数| 在字节中，无需转换操作的大小。
transog.earliest_last_modified_age| 整数| 最早的翻译年龄。
Translog.Remote_store| 目的| 与远程转换商店的操作有关的统计信息。
Translog.remote_store.upload| 目的| 统计信息与上传到远程转换商店有关。
Translog.remote_store.upload.total_uploads| 目的| 远程转换商店的同步数。
Translog.remote_store.upload.total_uploads.started| 整数| 上载同步到已启动的远程转换存储的数量。
Translog.remote_store.upload.total_uploads.failed| 整数| 上载同步到远程转换商店的失败的数量。
Translog.remote_store.upload.total_uploads.succeeded| 整数| 成功上传同步到远程转换商店的数量。
transog.remote_store.upload.total_upload_size| 目的| 上传到远程转换商店的数据总量。
Translog.remote_store.upload.total_upload_size.started_bytes| 整数| 上传启动后，字节的数量会积极上传到远程转换商店。
Translog.remote_store.upload.total_upload_size.failed_bytes| 整数| 未能上传到远程翻译存储的字节数。
Translog.remote_store.upload.total_upload_size.succeeded_bytes| 整数| 字节的数量成功上传到了远程转换商店。
request_cache| 目的| 有关该节点请求缓存的统计信息。
request_cache.memory_size_in_bytes| 整数| 请求缓存使用的内存大小在字节中。
request_cache.evictions| 整数| 请求缓存驱逐的数量。
request_cache.hit_count| 整数| 请求缓存命中次数。
request_cache.miss_count| 整数| 请求缓存的数量错过。
恢复| 目的| 有关节点恢复操作的统计信息。
recovery.current_as_source| 整数| 使用索引碎片作为来源的恢复操作的数量。
recovery.current_as_target| 整数| 使用索引碎片作为目标的恢复操作的数量。
recovery.throttle_time_in_millis| 整数| 恢复操作由于节流而延迟，以毫秒为单位。


### `os`

这`os` 对象具有该节点的OS统计信息，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
时间戳| 整数| OS统计数据的最后一次刷新时间是自时代以来的毫秒。
中央处理器| 目的| 有关该节点CPU用法的统计数据。
CPU| 整数| 该系统最近使用CPU。
cpu.load_average| 目的| 有关系统的负载平均值的统计信息。
cpu.load_average.1m| 漂浮| 一分钟时间内系统的负载平均值。
cpu.load_average.5m| 漂浮| 五分钟时间内系统的负载平均值。
cpu.load_average.15m| 漂浮| 15分钟时间内系统的负载平均值。
cpu.mem| 目的| 有关节点的内存使用量的统计信息。
cpu.mem.total_in_bytes| 整数| 物理记忆的总数，字节。
cpu.mem.free_in_bytes| 整数| 主字节的免费物理内存总量。
cpu.mem.used_in_bytes| 整数| 使用的物理内存的总量，字节。
cpu.mem.free_percent| 整数| 免费内存的百分比。
cpu.mem.used_percent| 整数| 使用的内存百分比。
cpu.swap| 目的| 有关节点交换空间的统计信息。
cpu.swap.total_in_bytes| 整数| 交换空间的总量，字节。
cpu.swap.free_in_bytes| 整数| 自由交换空间的总量，字节。
cpu.swap.used_in_bytes| 整数| 使用的交换空间的总量，字节。
cpu.cgroup| 目的| 包含该节点的cgroup统计信息。仅返回Linux。
cpu.cgroup.cpuacct| 目的| 有关节点的CPUACCT对照组的统计信息。
cpu.cgroup.cpu| 目的| 有关该节点的CPU对照组的统计信息。
cpu.cgroup.memory| 目的| 有关节点的内存控制组的统计信息。

### `process`

这`process` 对象包含该节点的过程统计信息，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
时间戳| 整数| 该过程统计的最后一次刷新时间是自时代以来的毫秒。
Open_FILE_DESCRIPTOR| 整数|  与当前过程关联的打开文件描述符的数量。
max_file_descriptors| 整数| 系统的最大文件描述符数。
中央处理器| 目的| 有关该节点的CPU的统计信息。
CPU| 整数| 该过程的CPU使用百分比。
cpu.total_in_millis| 整数| JVM以毫秒为单位运行的过程所使用的总CPU时间。
mem| 目的| 有关节点内存的统计信息。
mem.total_virtual_in_bytes| 整数| 保证可用于当前正在运行的流程的虚拟内存总量，以字节为单位。


### `jvm`

这`jvm` 对象包含有关该节点的JVM的统计信息，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
时间戳| 整数| JVM统计数据的最后一次刷新时间是自时代以来的毫秒。
正常时间_in_millis| 整数| JVM正常运行时间，以毫秒为单位。
mem| 目的| 节点上的JVM内存使用量的统计信息。
mem.heap_used_in_bytes| 整数| 当前使用的内存量，字节中。
mem.heap_used_percent| 整数| 堆目前使用的内存百分比。
mem.heap_committy_in_bytes| 整数| 堆可用的内存量，字节中。
mem.heap_max_in_bytes| 整数| 堆可用的最大内存量，以字节为单位。
mem.non_heap_used_in_bytes| 整数| 非数量-当前使用的堆内存在字节中。
mem.non_heap_committed_in_bytes| 整数| 最大非-可在字节中使用的堆内存。
mem.pools| 目的| 有关该节点的堆内存使用的统计信息。
mem.pools.young| 目的| 有关该节点的年轻一代堆内存使用的统计数据。包含所使用的内存量，可用的最大内存量以及所使用的内存峰值。
mem.pools.old| 目的| 有关该节点的旧一代堆内存使用的统计信息。包含所使用的内存量，可用的最大内存量以及所使用的内存峰值。
mem.pools.survivor| 目的| 有关该节点的幸存者空间内存使用量的统计信息。包含所使用的内存量，可用的最大内存量以及所使用的内存峰值。
线程| 目的| 有关该节点的JVM线程使用情况的统计信息。
线程| 整数| JVM中当前活动的线程数量。
螺纹。Peak_count| 整数| JVM中的最大线程数。
GC.Collectors| 目的| 有关节点的JVM垃圾收集器的统计数据。
gc.collectors.young| 整数| 关于收集年轻一代物体的JVM垃圾收集器的统计数据。
gc.collectors.young.collection_count| 整数| 收集年轻一代物体的垃圾收集器数量。
gc.collectors.young.collection_time_in_millis| 整数| 以毫秒为单位的年轻一代物体垃圾收集的总时间。
gc.collectors.old| 整数| 关于收集旧一代对象的JVM垃圾收集器的统计数据。
gc.collectors.old.collection_count| 整数| 收集旧一代物体的垃圾收集器数量。
gc.collectors.old.collection_time_in_millis| 整数| 以毫秒为单位的垃圾收集垃圾收集的总时间。
buffer_pools| 目的| 有关节点的JVM缓冲池的统计信息。
buffer_pools.mapped| 目的| 有关节点的映射JVM缓冲池的统计信息。
buffer_pools.mapped.count| 整数| 映射的缓冲池的数量。
buffer_pools.mapped.used_in_bytes| 整数| 映射的缓冲池使用的内存量，字节中。
buffer_pools.mapped.total_capacity_in_bytes| 整数| 映射的缓冲池的总容量，字节。
buffer_pools.direct| 目的| 有关节点的直接JVM缓冲池的统计信息。
buffer_pools.direct.count| 整数| 直接缓冲池的数量。
buffer_pools.direct.used_in_bytes| 整数| 直接缓冲池使用的内存量，字节。
buffer_pools.direct.total_capacity_in_bytes| 整数| 直接缓冲池的总容量，字节。
课程| 目的| 关于JVM为节点加载的类的统计信息。
class.current_loaded_count| 整数| JVM当前加载的类数量。
class.total_loaded_count| 整数| JVM开始加载的类总数。
class.total_unloaded_count| 整数| 自JVM启动以来，JVM卸载的类总数。

### `thread_pool`

这`thread_pool` 对象包含所有线程池的列表。每个线程池是一个由ID指定的嵌套对象，并包含以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
线程| 整数| 池中的线程数。
队列| 整数| 队列中的线程数。
积极的| 整数| 池中的活动线数量。
拒绝| 整数| 被拒绝的任务数量。
最大| 整数| 池中线的峰值数。
完全的| 整数| 完成的任务数量。
total_wait_time| 整数| 在线程池队列中等待的总时间任务总数。目前，只有`search`，`search_throttled`， 和`index_searcher` 线程池支持此指标。

### `fs`

这`fs` 对象表示有关该节点的文件存储的统计信息。它具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
时间戳| 整数| 文件存储统计信息的最后一次刷新时间是自时代以来的毫秒。
全部的| 目的| 节点所有文件存储的统计信息。
total.total_in_bytes| 整数| 所有文件存储的总内存大小，字节中。
total.free_in_bytes| 整数| 所有文件存储中的总未分配磁盘空间，字节中。
total.available_in_bytes| 整数| 所有文件存储的JVM可用的总磁盘空间。代表OpenSearch可以使用的实际内存数量，在字节中。
数据| 大批| 所有文件存储的列表。每个文件存储都有以下属性。
数据| 细绳| 文件存储的路径。
data.mount| 细绳| 文件存储的安装点。
数据类型| 细绳| 文件存储的类型（例如，覆盖）。
data.total_in_bytes| 整数| 文件存储的总大小，字节。
data.free_in_bytes| 整数| 文件存储中的总未分配磁盘空间，字节。
data.available_in_bytes| 整数| JVM可用于文件存储的磁盘空间总数，字节。
io_stats| 目的| 节点的I/O统计信息（仅Linux）。包括设备，读写操作以及I/O操作时间。

### `transport`

这`transport` 对象具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
server_open| 整数| OpenSearch节点用于内部通信的开放入站TCP连接的数量。
total_outbound_connect| 整数| 自节点启动以来，该节点已经打开的出站传输连接总数。
rx_count| 整数| 内部通信期间收到的节点收到的RX（接收）数据包的总数。
rx_size_in_bytes| 整数| 内部通信期间收到的RX数据包的总大小，字节。
TX_COUNT| 整数| 在内部通信期间发送的节点发送的TX（发送）数据包的总数。
tx_size_in_bytes| 整数| 在内部通信期间发送的节点在字节中发送的TX（传输）数据包的总大小。

### `breakers`

这`breakers` 对象包含有关该节点断路器的统计信息。每个断路器是按名称列出的嵌套对象，并包含以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
limit_size_in_bytes| 整数| 断路器的内存限制，字节。
limit_size| 字节值| 人类断路器的记忆限制-可读格式（例如，`307.1mb`）。
estunated_size_in_bytes| 整数| 用于操作的估计内存，以字节为单位。
estated_size| 字节值| 人类操作的估计记忆-可读格式（例如，`356b`）。
高架| 漂浮| 所有估计值都乘以计算最终估计值的因素。
绊倒| 整数| 断路器已激活的总数以防止-的-内存错误。

### `script` 和`script_cache`

这`script` 和`script_cache` 对象具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
脚本| 目的| 节点的脚本统计信息。
script.com| 整数| 节点的脚本汇编总数。
script.cache_evictions| 整数| 脚本缓存已清除旧数据的总数。
script.compilation_limit_trigged| 整数| 脚本编译的总数受到断路器的限制。
script_cache| 目的| 节点的脚本缓存统计信息。
script_cache.sum.compilations| 整数| 节点缓存中脚本汇编的总数。
script_cache.sum.cache_evictions| 整数| 脚本缓存已清除旧数据的总数。
script_cache.sum.compilation_limit_trigged| 整数| 缓存中脚本编译的总数受到断路器的限制。
script_cache.contexts| 对象数组| 脚本缓存的上下文列表。每个上下文都包含其名称，汇编数量，缓存驱逐的数量以及脚本受到断路器限制的次数。

### `discovery`

这`discovery` 对象包含节点发现统计信息，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
cluster_state_queue| 目的| 该节点的群集状态队列统计。
cluster_state_queue.total| 整数| 队列中的群集状态总数。
cluster_state_queue.perding| 整数| 队列中待处理的群集的数量。
cluster_state_queue.committ| 整数| 队列中授权的集群状态的数量。
uplanced_cluster_states| 目的| 该节点已发表的群集状态的统计信息。
upplyed_cluster_states.full_states| 整数| 已发表的集群状态数量。
uppared_cluster_states.incompatible_diffs| 整数| 已发表的群集状态之间不兼容的差异的数量。
publined_cluster_states.compatible_diffs| 整数| 已发表的集群状态之间兼容差异的数量。

### `ingest`

这`ingest` 对象包含摄入统计信息，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
全部的| 整数| 节点寿命的摄入统计信息。
总数| 整数| 节点摄入的文档总数。
total.time_in_millis| 整数| 预处理摄入文件的总时间，以毫秒为单位。
总计| 整数| 该节点当前正在摄入的文档总数。
总计| 整数| 该节点的摄入量失败总数。
管道| 目的| 节点的摄入管道统计。每个管道都是由其ID指定的嵌套对象，并具有以下属性。
pipelines._id_.count| 整数| 摄入管道预处理的文档数量。
pipelines._id_.time_in_millis| 整数| 以毫秒为单位的摄入管道中预处理文档的总时间。
pipelines._id_.failed| 整数| 摄入管道的摄入量失败总数。
pipelines._id_.processors| 对象数组| 摄入处理器的统计数据。包括当前转换的文档数量，转换文档的总数，失败的转换数量以及所花费的时间转换文档。

### `search_pipeline`

这`search_pipeline` 对象包含与[搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/) 并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
total_request| 目的| 与所有搜索请求处理器有关的累积统计信息。
total_request.count| 整数| 搜索请求处理器执行的总数。
total_request.time_in_millis| 整数| 所有搜索请求处理器执行的总时间（以毫秒为单位）。
total_request.current| 整数| 当前正在进行的搜索请求处理器执行总数。
total_request.failed| 整数| 搜索请求处理器执行的失败总数。
total_response| 目的| 与所有搜索响应处理器有关的累积统计数据。
total_response.count| 整数| 搜索响应处理器执行的总数。
total_response.time_in_millis| 整数| 所有搜索响应处理器执行的总时间，以毫秒为单位。
total_response.current| 整数| 当前正在进行的搜索响应处理器执行总数。
total_response.failed| 整数| 搜索响应处理器执行的失败总数。
管道| 目的| 搜索管道统计。每个管道都是由其ID指定的嵌套对象，并在以下行中列出了属性。如果处理器有一个`tag`，处理器的统计信息在对象中提供了名称`<processor_type>:<tag>` （例如，`filter_query:abc`）。所有相同类型的所有处理器的统计数据`tag` 汇总并在对象中提供名称`<processor-type>` （例如，`filter_query`）。
pipelines._id_.request.count| 整数| 搜索管道执行的搜索请求处理器的执行次数。
pipelines._id_.request.time_in_millis| 整数| 搜索管道中搜索请求处理器执行的总时间，以毫秒为单位。
pipelines._id_.request.current| 整数| 搜索管道当前正在进行的搜索请求处理器执行次数。
pipelines._id_.request.failed| 整数| 搜索管道的搜索请求处理器执行的失败数量。
pipelines._id_.request_processors| 对象数组| 搜索请求处理器的统计信息。包括执行总数，执行时间总数，当前正在进行的执行总数以及失败的执行次数。
pipelines._id_.response.count| 整数| 搜索管道执行的搜索响应处理器执行次数。
pipelines._id_.response.time_in_millis| 整数| 搜索管道中搜索响应处理器执行的总时间，以毫秒为单位。
pipelines._id_.response.current| 整数| 搜索管道当前正在进行的搜索响应处理器执行次数。
pipelines._id_.response.failed| 整数| 搜索管道的搜索响应处理器执行的失败数量。
pipelines._id_.response_processors| 对象数组| 搜索响应处理器的统计信息。包括执行总数，执行时间总数，当前正在进行的执行总数以及失败的执行次数。

### `adaptive_selection`

这`adaptive_selection` 对象包含自适应选择统计信息。每个条目由节点ID指定，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
外卖_SEREARZES| 整数| 该节点的传出搜索请求数量。
avg_queue_size| 整数| 节点的搜索请求的滚动平均队列大小（指数加权）。
avg_service_time_ns| 整数| 搜索请求的滚动平均服务时间，纳秒（指数加权）。
avg_response_time_ns| 整数| 搜索请求的滚动平均响应时间，纳秒（指数加权）。
秩| 细绳| 路由请求时用于选择碎片的节点等级。

### `indexing_pressure`

这`indexing_pressure` 对象包含索引压力统计，并具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
记忆| 目的| 与索引负载的内存消耗有关的统计信息。
内存| 目的| 与当前索引负载的内存消耗有关的统计信息。
memory.current.combined_coordinating_and_primary_in_bytes| 整数| 在字节中，在协调或主要阶段中索引请求使用的总内存。如果主阶段在本地运行，则节点可以重复使用协调内存，因此总存储器不一定等于协调和主要阶段内存使用的总和。
memory.current.coordinating_in_bytes| 在协调阶段的字节中，通过索引请求消耗的总内存。
memory.current.primary_in_bytes| 整数| 在主阶段，字节中，通过索引请求消耗的总内存。
memory.current.replica_in_bytes| 整数| 在复制阶段，字节中，通过索引请求消耗的总内存。
memory.current.all_in_bytes| 整数| 通过在协调，主或复制阶段的索引请求消耗的总内存。

### `shard_indexing_pressure`

这`shard_indexing_pressure` 对象包含[碎片索引压力]({{site.url}}{{site.baseurl}}/opensearch/shard-indexing-backpressure) 统计数据，具有以下属性。

场地| 字段类型| 描述
:--- | :--- | :---
[统计]({{site.url}}{{site.baseurl}}/opensearch/stats-api/) | 目的| 有关碎片索引压力的统计数据。
total_rejections_breakup_shadow_mode| 目的| 如果在影子模式下运行`total_rejections_breakup_shadow_mode` 对象包含有关节点中所有碎片的请求拒绝标准的统计信息。
total_rejections_breakup_shadow_mode.node_limits| 整数| 由于节点内存限制而引起的拒绝总数。当所有碎片达到分配给节点的内存限制（例如，堆尺寸的10％）时，碎片无法吸收更多的节点流量，并且索引请求被拒绝。
total_rejections_breakup_shadow_mode.no_successful_request_limits| 整数| 当节点占用水平违反其软限制时，拒绝总数和碎片有多个未偿还的请求，等待执行。在这种情况下，在系统恢复之前拒绝其他索引请求。
total_rejections_breakup_shadow_mode.throughput_degradation_limits| 整数| 当节点占用水平违反其软限制时，拒绝总数且在碎片级别的请求周转中存在恒定的恶化。在这种情况下，在系统恢复之前拒绝其他索引请求。
启用| 布尔| 指定是否打开节点的碎片索引压力特征。
执行| 布尔| 如果是真的，则碎片索引压力在强制模式下运行（有拒绝）。如果是错误的，则碎片索引压力在阴影模式下运行（没有拒绝，但是记录了统计信息，可以在`total_rejections_breakup_shadow_mode` 目的）。仅当启用碎片索引压力时才适用。

## 并发段搜索

从OpenSearch 2.10开始[并发段搜索]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search/) 允许每个碎片-在查询阶段并行搜索段的级别请求。如果你[启用实验并发段搜索功能标志]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search#enabling-the-feature-flag)，节点统计API响应将包含一些其他字段，其中包含有关切片的统计信息（线程执行的工作单位）。有关这些字段的描述，请参阅[索引统计API]({{site.url}}{{site.baseurl}}/api-reference/index-apis/stats#concurrent-segment-search)。

## 所需的权限

如果使用安全插件，请确保拥有适当的权限：`cluster:monitor/nodes/stats`。


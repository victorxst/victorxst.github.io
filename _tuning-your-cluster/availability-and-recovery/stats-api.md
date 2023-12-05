---
layout: default
title: 统计API
parent: 碎片索引背压
nav_order: 2
grand_parent: 可用性和恢复
has_children: false
redirect_from: 
  - /opensearch/stats-api/
---

# 统计API

使用统计操作监视shard索引背压。

## 统计
引入了1.2
{: .label .label-purple }

返回节点-水平和碎片-索引请求拒绝的级别统计数据。

#### 要求

```json
GET _nodes/_local/stats/shard_indexing_pressure
```

如果`enforced` 是`true`：

#### 示例响应

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "q3e1dQjFSqyPSLAgpyQlfw": {
      "timestamp": 1613072111162,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
        "data",
        "ingest",
        "cluster_manager",
        "remote_cluster_client"
      ],
      "attributes": {
        "testattr": "test"
      },
      "shard_indexing_pressure": {
        "stats": {
          "[index_name][0]": {
            "memory": {
              "current": {
                "coordinating_in_bytes": 0,
                "primary_in_bytes": 0,
                "replica_in_bytes": 0
              },
              "total": {
                "coordinating_in_bytes": 299,
                "primary_in_bytes": 299,
                "replica_in_bytes": 0
              }
            },
            "rejection": {
              "coordinating": {
                "coordinating_rejections": 0,
                "breakup": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              },
              "primary": {
                "primary_rejections": 0,
                "breakup": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              },
              "replica": {
                "replica_rejections": 0,
                "breakup": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              }
            },
            "last_successful_timestamp": {
              "coordinating_last_successful_request_timestamp_in_millis": 1613072107990,
              "primary_last_successful_request_timestamp_in_millis": 0,
              "replica_last_successful_request_timestamp_in_millis": 0
            },
            "indexing": {
              "coordinating_time_in_millis": 96,
              "coordinating_count": 1,
              "primary_time_in_millis": 0,
              "primary_count": 0,
              "replica_time_in_millis": 0,
              "replica_count": 0
            },
            "memory_allocation": {
              "current": {
                "current_coordinating_and_primary_bytes": 0,
                "current_replica_bytes": 0
              },
              "limit": {
                "current_coordinating_and_primary_limits_in_bytes": 51897,
                "current_replica_limits_in_bytes": 77845
              }
            }
          }
        },
        "total_rejections_breakup": {
          "node_limits": 0,
          "no_successful_request_limits": 0,
          "throughput_degradation_limits": 0
        },
        "enabled": true,
        "enforced" : true
      }
    }
  }
}
```

如果`enforced` 是`false`：

#### 示例响应

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "q3e1dQjFSqyPSLAgpyQlfw": {
      "timestamp": 1613072111162,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
        "data",
        "ingest",
        "cluster_manager",
        "remote_cluster_client"
      ],
      "attributes": {
        "testattr": "test"
      },
      "shard_indexing_pressure": {
        "stats": {
          "[index_name][0]": {
            "memory": {
              "current": {
                "coordinating_in_bytes": 0,
                "primary_in_bytes": 0,
                "replica_in_bytes": 0
              },
              "total": {
                "coordinating_in_bytes": 299,
                "primary_in_bytes": 299,
                "replica_in_bytes": 0
              }
            },
            "rejection": {
              "coordinating": {
                "coordinating_rejections": 0,
                "breakup_shadow_mode": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              },
              "primary": {
                "primary_rejections": 0,
                "breakup_shadow_mode": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              },
              "replica": {
                "replica_rejections": 0,
                "breakup_shadow_mode": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              }
            },
            "last_successful_timestamp": {
              "coordinating_last_successful_request_timestamp_in_millis": 1613072107990,
              "primary_last_successful_request_timestamp_in_millis": 0,
              "replica_last_successful_request_timestamp_in_millis": 0
            },
            "indexing": {
              "coordinating_time_in_millis": 96,
              "coordinating_count": 1,
              "primary_time_in_millis": 0,
              "primary_count": 0,
              "replica_time_in_millis": 0,
              "replica_count": 0
            },
            "memory_allocation": {
              "current": {
                "current_coordinating_and_primary_bytes": 0,
                "current_replica_bytes": 0
              },
              "limit": {
                "current_coordinating_and_primary_limits_in_bytes": 51897,
                "current_replica_limits_in_bytes": 77845
              }
            }
          }
        },
        "total_rejections_breakup_shadow_mode": {
          "node_limits": 0,
          "no_successful_request_limits": 0,
          "throughput_degradation_limits": 0
        },
        "enabled": true,
        "enforced" : false
      }
    }
  }
}
```

要包括所有在其上执行的活动和先前的写操作的碎片，请指定`include_all` 范围：

#### 要求

```json
GET _nodes/_local/stats/shard_indexing_pressure?include_all
```

#### 示例响应

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "q3e1dQjFSqyPSLAgpyQlfw": {
      "timestamp": 1613072198171,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
        "data",
        "ingest",
        "cluster_manager",
        "remote_cluster_client"
      ],
      "attributes": {
        "testattr": "test"
      },
      "shard_indexing_pressure": {
        "stats": {
          "[index_name][0]": {
            "memory": {
              "current": {
                "coordinating_in_bytes": 0,
                "primary_in_bytes": 0,
                "replica_in_bytes": 0
              },
              "total": {
                "coordinating_in_bytes": 604,
                "primary_in_bytes": 604,
                "replica_in_bytes": 0
              }
            },
            "rejection": {
              "coordinating": {
                "coordinating_rejections": 0,
                "breakup": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              },
              "primary": {
                "primary_rejections": 0,
                "breakup": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              },
              "replica": {
                "replica_rejections": 0,
                "breakup": {
                  "node_limits": 0,
                  "no_successful_request_limits": 0,
                  "throughput_degradation_limits": 0
                }
              }
            },
            "last_successful_timestamp": {
              "coordinating_last_successful_request_timestamp_in_millis": 1613072194656,
              "primary_last_successful_request_timestamp_in_millis": 0,
              "replica_last_successful_request_timestamp_in_millis": 0
            },
            "indexing": {
              "coordinating_time_in_millis": 145,
              "coordinating_count": 2,
              "primary_time_in_millis": 0,
              "primary_count": 0,
              "replica_time_in_millis": 0,
              "replica_count": 0
            },
            "memory_allocation": {
              "current": {
                "current_coordinating_and_primary_bytes": 0,
                "current_replica_bytes": 0
              },
              "limit": {
                "current_coordinating_and_primary_limits_in_bytes": 51897,
                "current_replica_limits_in_bytes": 77845
              }
            }
          }
        },
        "total_rejections_breakup": {
          "node_limits": 0,
          "no_successful_request_limits": 0,
          "throughput_degradation_limits": 0
        },
        "enabled": true,
        "enforced": true
      }
    }
  }
}
```

仅获得所有顶部-级别汇总属性，指定`top` 参数（跳过Per-碎片统计数据）。

#### 要求

```json
GET _nodes/_local/stats/shard_indexing_pressure?top
```

如果`enforced` 是`true`：

#### 示例响应

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "q3e1dQjFSqyPSLAgpyQlfw": {
      "timestamp": 1613072382719,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
        "data",
        "ingest",
        "cluster_manager",
        "remote_cluster_client"
      ],
      "attributes": {
        "testattr": "test"
      },
      "shard_indexing_pressure": {
        "stats": {},
        "total_rejections_breakup": {
          "node_limits": 0,
          "no_successful_request_limits": 0,
          "throughput_degradation_limits": 0
        },
        "enabled": true,
        "enforced": true
      }
    }
  }
}
```

如果`enforced` 是`false`：

#### 示例响应

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "q3e1dQjFSqyPSLAgpyQlfw": {
      "timestamp": 1613072382719,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
        "data",
        "ingest",
        "cluster_manager",
        "remote_cluster_client"
      ],
      "attributes": {
        "testattr": "test"
      },
      "shard_indexing_pressure": {
        "stats": {},
        "total_rejections_breakup_shadow_mode": {
          "node_limits": 0,
          "no_successful_request_limits": 0,
          "throughput_degradation_limits": 0
        },
        "enabled": true,
        "enforced" : false
      }
    }
  }
}
```

得到碎片-每个节点的拒绝级别分解（仅包括具有主动写操作的碎片）：

#### 要求

```json
GET _nodes/stats/shard_indexing_pressure
```

#### 示例响应

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "q3e1dQjFSqyPSLAgpyQlfw": {
      "timestamp": 1613072382719,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
        "data",
        "ingest",
        "cluster_manager",
        "remote_cluster_client"
      ],
      "attributes": {
        "testattr": "test"
      },
      "shard_indexing_pressure": {
        "stats": {},
        "total_rejections_breakup": {
          "node_limits": 0,
          "no_successful_request_limits": 0,
          "throughput_degradation_limits": 0
        },
        "enabled": true,
        "enforced": true
      }
    }
  }
}
```


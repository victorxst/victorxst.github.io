---
layout: default
title: 集群统计
nav_order: 60
parent: 群集API
has_children: false
redirect_from:
  - /api-reference/cluster-stats/
  - /opensearch/rest-api/cluster-stats/
---

# 集群统计
**引入1.0**
{: .label .label-purple }

群集统计API操作返回有关群集的统计信息。

## 例子

```json
GET _cluster/stats/nodes/_cluster_manager
```
{% include copy-curl.html %}

## 路径和HTTP方法

```json
GET _cluster/stats
GET _cluster/stats/nodes/<node-filters>
```

## URL参数

所有群集统计参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
＆lt; node-过滤器＆gt;| 列表| 逗号-分开的列表[节点过滤器]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/#node-filters) OpenSearch用来过滤结果。


  虽然`master` 现在称为节点`cluster_manager` 对于2.0版，我们保留了`master` 向后兼容的字段。如果您的节点有一个`master` 角色或`cluster_manager` 角色，`count` 这两个字段的增加。要查看示例节点计数的增加，请参见响应样本。
   {:.note}

## 回复

```json
{
    "_nodes": {
        "total": 1,
        "successful": 1,
        "failed": 0
    },
    "cluster_name": "opensearch-cluster",
    "cluster_uuid": "QravFieJS_SlZJyBMcDMqQ",
    "timestamp": 1644607845054,
    "status": "yellow",
    "indices": {
        "count": 114,
        "shards": {
            "total": 121,
            "primaries": 60,
            "replication": 1.0166666666666666,
            "index": {
                "shards": {
                    "min": 1,
                    "max": 2,
                    "avg": 1.0614035087719298
                },
                "primaries": {
                    "min": 0,
                    "max": 2,
                    "avg": 0.5263157894736842
                },
                "replication": {
                    "min": 0.0,
                    "max": 1.0,
                    "avg": 0.008771929824561403
                }
            }
        },
        "docs": {
            "count": 134263,
            "deleted": 115
        },
        "store": {
            "size_in_bytes": 70466547,
            "reserved_in_bytes": 0
        },
        "fielddata": {
            "memory_size_in_bytes": 664,
            "evictions": 0
        },
        "query_cache": {
            "memory_size_in_bytes": 0,
            "total_count": 1,
            "hit_count": 0,
            "miss_count": 1,
            "cache_size": 0,
            "cache_count": 0,
            "evictions": 0
        },
        "completion": {
            "size_in_bytes": 0
        },
        "segments": {
            "count": 341,
            "memory_in_bytes": 3137244,
            "terms_memory_in_bytes": 2488992,
            "stored_fields_memory_in_bytes": 167672,
            "term_vectors_memory_in_bytes": 0,
            "norms_memory_in_bytes": 346816,
            "points_memory_in_bytes": 0,
            "doc_values_memory_in_bytes": 133764,
            "index_writer_memory_in_bytes": 0,
            "version_map_memory_in_bytes": 0,
            "fixed_bit_set_memory_in_bytes": 1112,
            "max_unsafe_auto_id_timestamp": 1644269449096,
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
        "mappings": {
            "field_types": [
                {
                    "name": "alias",
                    "count": 1,
                    "index_count": 1
                },
                {
                    "name": "binary",
                    "count": 1,
                    "index_count": 1
                },
                {
                    "name": "boolean",
                    "count": 87,
                    "index_count": 22
                },
                {
                    "name": "date",
                    "count": 185,
                    "index_count": 91
                },
                {
                    "name": "double",
                    "count": 5,
                    "index_count": 2
                },
                {
                    "name": "float",
                    "count": 4,
                    "index_count": 1
                },
                {
                    "name": "geo_point",
                    "count": 4,
                    "index_count": 3
                },
                {
                    "name": "half_float",
                    "count": 12,
                    "index_count": 1
                },
                {
                    "name": "integer",
                    "count": 144,
                    "index_count": 29
                },
                {
                    "name": "ip",
                    "count": 2,
                    "index_count": 1
                },
                {
                    "name": "keyword",
                    "count": 1939,
                    "index_count": 109
                },
                {
                    "name": "knn_vector",
                    "count": 1,
                    "index_count": 1
                },
                {
                    "name": "long",
                    "count": 158,
                    "index_count": 92
                },
                {
                    "name": "nested",
                    "count": 25,
                    "index_count": 10
                },
                {
                    "name": "object",
                    "count": 420,
                    "index_count": 91
                },
                {
                    "name": "text",
                    "count": 1768,
                    "index_count": 102
                }
            ]
        },
        "analysis": {
            "char_filter_types": [],
            "tokenizer_types": [],
            "filter_types": [],
            "analyzer_types": [],
            "built_in_char_filters": [],
            "built_in_tokenizers": [],
            "built_in_filters": [],
            "built_in_analyzers": [
                {
                    "name": "english",
                    "count": 1,
                    "index_count": 1
                }
            ]
        }
    },
    "nodes": {
        "count": {
            "total": 1,
            "coordinating_only": 0,
            "data": 1,
            "ingest": 1,
            "master": 1,
            "cluster_manager": 1,
            "remote_cluster_client": 1
        },
        "versions": [
            "1.2.4"
        ],
        "os": {
            "available_processors": 6,
            "allocated_processors": 6,
            "names": [
                {
                    "name": "Linux",
                    "count": 1
                }
            ],
            "pretty_names": [
                {
                    "pretty_name": "Amazon Linux 2",
                    "count": 1
                }
            ],
            "mem": {
                "total_in_bytes": 6232674304,
                "free_in_bytes": 1452658688,
                "used_in_bytes": 4780015616,
                "free_percent": 23,
                "used_percent": 77
            }
        },
        "process": {
            "cpu": {
                "percent": 0
            },
            "open_file_descriptors": {
                "min": 970,
                "max": 970,
                "avg": 970
            }
        },
        "jvm": {
            "max_uptime_in_millis": 108800629,
            "versions": [
                {
                    "version": "15.0.1",
                    "vm_name": "OpenJDK 64-Bit Server VM",
                    "vm_version": "15.0.1+9",
                    "vm_vendor": "AdoptOpenJDK",
                    "bundled_jdk": true,
                    "using_bundled_jdk": true,
                    "count": 1
                }
            ],
            "mem": {
                "heap_used_in_bytes": 178956256,
                "heap_max_in_bytes": 536870912
            },
            "threads": 112
        },
        "fs": {
            "total_in_bytes": 62725623808,
            "free_in_bytes": 28442726400,
            "available_in_bytes": 25226010624
        },
        "plugins": [
            {
                "name": "opensearch-index-management",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch Index Management Plugin",
                "classname": "org.opensearch.indexmanagement.IndexManagementPlugin",
                "custom_foldername": "",
                "extended_plugins": [
                    "opensearch-job-scheduler"
                ],
                "has_native_controller": false
            },
            {
                "name": "opensearch-security",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "Provide access control related features for OpenSearch 1.0.0",
                "classname": "org.opensearch.security.OpenSearchSecurityPlugin",
                "custom_foldername": "opensearch-security",
                "extended_plugins": [],
                "has_native_controller": false
            },
            {
                "name": "opensearch-cross-cluster-replication",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch Cross Cluster Replication Plugin",
                "classname": "org.opensearch.replication.ReplicationPlugin",
                "custom_foldername": "",
                "extended_plugins": [],
                "has_native_controller": false
            },
            {
                "name": "opensearch-job-scheduler",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch Job Scheduler plugin",
                "classname": "org.opensearch.jobscheduler.JobSchedulerPlugin",
                "custom_foldername": "",
                "extended_plugins": [],
                "has_native_controller": false
            },
            {
                "name": "opensearch-anomaly-detection",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch anomaly detector plugin",
                "classname": "org.opensearch.ad.AnomalyDetectorPlugin",
                "custom_foldername": "",
                "extended_plugins": [
                    "lang-painless",
                    "opensearch-job-scheduler"
                ],
                "has_native_controller": false
            },
            {
                "name": "opensearch-performance-analyzer",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch Performance Analyzer Plugin",
                "classname": "org.opensearch.performanceanalyzer.PerformanceAnalyzerPlugin",
                "custom_foldername": "",
                "extended_plugins": [],
                "has_native_controller": false
            },
            {
                "name": "opensearch-reports-scheduler",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "Scheduler for Dashboards Reports Plugin",
                "classname": "org.opensearch.reportsscheduler.ReportsSchedulerPlugin",
                "custom_foldername": "",
                "extended_plugins": [
                    "opensearch-job-scheduler"
                ],
                "has_native_controller": false
            },
            {
                "name": "opensearch-asynchronous-search",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "Provides support for asynchronous search",
                "classname": "org.opensearch.search.asynchronous.plugin.AsynchronousSearchPlugin",
                "custom_foldername": "",
                "extended_plugins": [],
                "has_native_controller": false
            },
            {
                "name": "opensearch-knn",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch k-NN plugin",
                "classname": "org.opensearch.knn.plugin.KNNPlugin",
                "custom_foldername": "",
                "extended_plugins": [
                    "lang-painless"
                ],
                "has_native_controller": false
            },
            {
                "name": "opensearch-alerting",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "Amazon OpenSearch alerting plugin",
                "classname": "org.opensearch.alerting.AlertingPlugin",
                "custom_foldername": "",
                "extended_plugins": [
                    "lang-painless"
                ],
                "has_native_controller": false
            },
            {
                "name": "opensearch-observability",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch Plugin for OpenSearch Dashboards Observability",
                "classname": "org.opensearch.observability.ObservabilityPlugin",
                "custom_foldername": "",
                "extended_plugins": [],
                "has_native_controller": false
            },
            {
                "name": "opensearch-sql",
                "version": "1.2.4.0",
                "opensearch_version": "1.2.4",
                "java_version": "1.8",
                "description": "OpenSearch SQL",
                "classname": "org.opensearch.sql.plugin.SQLPlugin",
                "custom_foldername": "",
                "extended_plugins": [],
                "has_native_controller": false
            }
        ],
        "network_types": {
            "transport_types": {
                "org.opensearch.security.ssl.http.netty.SecuritySSLNettyTransport": 1
            },
            "http_types": {
                "org.opensearch.security.http.SecurityHttpServerTransport": 1
            }
        },
        "discovery_types": {
            "zen": 1
        },
        "packaging_types": [
            {
                "type": "tar",
                "count": 1
            }
        ],
        "ingest": {
            "number_of_pipelines": 0,
            "processor_stats": {}
        }
    }
}
```

## 响应身体场

场地| 描述
:--- | :---
节点| 响应中返回了多少个节点。
cluster_name| 集群的名称。
cluster_uuid| 集群的UUID。
时间戳| 群集最后一次刷新的Unix时期时间。
地位| 集群的健康状况。
指数| 有关集群中索引的统计信息。
indices.count| 集群中有多少个索引。
索引| 有关集群碎片的信息。
indices.docs| 集群中仍有多少个文档，并删除了多少个文档。
indices.Store| 有关集群存储的信息。
indices.fielddata| 有关集群字段数据的信息
indices.query_cache| 有关集群查询缓存的数据。
indices.completion| 内存中有多少个字节来完成操作。
索引| 有关集群细分市场的信息，这些细分是小的Lucene索引。
索引| 集群中的映射。
指数。分析| 有关集群中使用的分析仪的信息。
节点| 有关群集中节点的统计信息。
nodes.count| 从请求返回了多少个节点。
nodes.versions| OpenSearch的版本编号。
nodes.os| 有关节点中使用的操作系统的信息。
Nodes.process| 返回的节点使用的过程。
nodes.jvm| 有关Java虚拟机的统计信息。
nodes.fs| 节点的文件存储。
Nodes.plugins| 集成在节点中的OpenSearch插件。
nodes.network_types| 节点内的传输和HTTP网络。
nodes.discovery_type| 节点用来在群集中找到其他节点的方法。
nodes.packaging_types| 有关节点的搜索分布的信息。
节点| 有关节点的摄入管道/节点的信息，如果有的话。
total_time_spent| 从远程商店下载或上传时，在集群中所有碎片的下载和上传时间的总数总数。


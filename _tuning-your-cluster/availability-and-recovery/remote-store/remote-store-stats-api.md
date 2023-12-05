---
layout: default
title: 远程商店统计API
nav_order: 20
parent: 远程支持存储
grand_parent: 可用性和恢复
---

# 远程商店统计API

引入2.8
{：.label .label-紫色的 }

使用远程存储统计API监视碎片-级遥控器-支持的存储性能。

从此API返回的指标仅与存储在远程上的索引有关-支持节点。对于在节点或群集级别的索引上的汇总输出，请使用[索引统计]({{site.url}}{{site.baseurl}}/api-reference/index-apis/stats/)，，，，[节点统计]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-stats/)， 或者[集群统计]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-stats/) API。

## 路径和HTTP方法

```json
GET _remotestore/stats/<index_name>
GET _remotestore/stats/<index_name>/<shard_id>
```

## 路径参数

下表列出了可用路径参数。所有路径参数都是可选的。

范围| 类型| 描述
：--- | ：--- | ：---
`index_name` | 细绳| 索引名称或索引模式。
`shard_id` | 细绳| 碎片ID。

## 索引的远程商店统计数据

使用以下API获取所有索引碎片的远程存储统计信息。

#### 示例请求

```json
GET _remotestore/stats/<index_name>
```
{％包含副本-curl.html％}

#### 示例响应

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-delta}

```json
{
    "_shards": {
        "total": 4,
        "successful": 4,
        "failed": 0
    },
    "indices": {
        "remote-index": {
            "shards": {
                "0": [{
                        "routing": {
                            "state": "STARTED",
                            "primary": true,
                            "node": "q1VxWZnCTICrfRc2bRW3nw"
                        },
                        "segment": {
                            "download": {},
                            "upload": {
                                "local_refresh_timestamp_in_millis": 1694171634102,
                                "remote_refresh_timestamp_in_millis": 1694171634102,
                                "refresh_time_lag_in_millis": 0,
                                "refresh_lag": 0,
                                "bytes_lag": 0,
                                "backpressure_rejection_count": 0,
                                "consecutive_failure_count": 0,
                                "total_uploads": {
                                    "started": 5,
                                    "succeeded": 5,
                                    "failed": 0
                                },
                                "total_upload_size": {
                                    "started_bytes": 15342,
                                    "succeeded_bytes": 15342,
                                    "failed_bytes": 0
                                },
                                "remote_refresh_size_in_bytes": {
                                    "last_successful": 0,
                                    "moving_avg": 3068.4
                                },
                                "upload_speed_in_bytes_per_sec": {
                                    "moving_avg": 99988.2
                                },
                                "remote_refresh_latency_in_millis": {
                                    "moving_avg": 44.0
                                }
                            }
                        },
                        "translog": {
                            "upload": {
                                "last_successful_upload_timestamp": 1694171633644,
                                "total_uploads": {
                                    "started": 6,
                                    "failed": 0,
                                    "succeeded": 6
                                },
                                "total_upload_size": {
                                    "started_bytes": 1932,
                                    "failed_bytes": 0,
                                    "succeeded_bytes": 1932
                                },
                                "total_upload_time_in_millis": 21478,
                                "upload_size_in_bytes": {
                                    "moving_avg": 322.0
                                },
                                "upload_speed_in_bytes_per_sec": {
                                    "moving_avg": 2073.8333333333335
                                },
                                "upload_time_in_millis": {
                                    "moving_avg": 3579.6666666666665
                                }
                            },
                            "download": {}
                        }
                    },
                    {
                        "routing": {
                            "state": "STARTED",
                            "primary": false,
                            "node": "EZuen5Y5Sv-eDCLwh9gv-Q"
                        },
                        "segment": {
                            "download": {
                                "last_sync_timestamp": 1694171634148,
                                "total_download_size": {
                                    "started_bytes": 15112,
                                    "succeeded_bytes": 15112,
                                    "failed_bytes": 0
                                },
                                "download_size_in_bytes": {
                                    "last_successful": 2910,
                                    "moving_avg": 1259.3333333333333
                                },
                                "download_speed_in_bytes_per_sec": {
                                    "moving_avg": 382387.3333333333
                                }
                            },
                            "upload": {}
                        },
                        "translog": {
                            "upload": {},
                            "download": {}
                        }
                    }
                ],
                "1": [{
                        "routing": {
                            "state": "STARTED",
                            "primary": false,
                            "node": "q1VxWZnCTICrfRc2bRW3nw"
                        },
                        "segment": {
                            "download": {
                                "last_sync_timestamp": 1694171633181,
                                "total_download_size": {
                                    "started_bytes": 18978,
                                    "succeeded_bytes": 18978,
                                    "failed_bytes": 0
                                },
                                "download_size_in_bytes": {
                                    "last_successful": 325,
                                    "moving_avg": 1265.2
                                },
                                "download_speed_in_bytes_per_sec": {
                                    "moving_avg": 456047.6666666667
                                }
                            },
                            "upload": {}
                        },
                        "translog": {
                            "upload": {},
                            "download": {}
                        }
                    },
                    {
                        "routing": {
                            "state": "STARTED",
                            "primary": true,
                            "node": "EZuen5Y5Sv-eDCLwh9gv-Q"
                        },
                        "segment": {
                            "download": {},
                            "upload": {
                                "local_refresh_timestamp_in_millis": 1694171633122,
                                "remote_refresh_timestamp_in_millis": 1694171633122,
                                "refresh_time_lag_in_millis": 0,
                                "refresh_lag": 0,
                                "bytes_lag": 0,
                                "backpressure_rejection_count": 0,
                                "consecutive_failure_count": 0,
                                "total_uploads": {
                                    "started": 6,
                                    "succeeded": 6,
                                    "failed": 0
                                },
                                "total_upload_size": {
                                    "started_bytes": 19208,
                                    "succeeded_bytes": 19208,
                                    "failed_bytes": 0
                                },
                                "remote_refresh_size_in_bytes": {
                                    "last_successful": 0,
                                    "moving_avg": 3201.3333333333335
                                },
                                "upload_speed_in_bytes_per_sec": {
                                    "moving_avg": 109612.0
                                },
                                "remote_refresh_latency_in_millis": {
                                    "moving_avg": 25.333333333333332
                                }
                            }
                        },
                        "translog": {
                            "upload": {
                                "last_successful_upload_timestamp": 1694171633106,
                                "total_uploads": {
                                    "started": 7,
                                    "failed": 0,
                                    "succeeded": 7
                                },
                                "total_upload_size": {
                                    "started_bytes": 2405,
                                    "failed_bytes": 0,
                                    "succeeded_bytes": 2405
                                },
                                "total_upload_time_in_millis": 27748,
                                "upload_size_in_bytes": {
                                    "moving_avg": 343.57142857142856
                                },
                                "upload_speed_in_bytes_per_sec": {
                                    "moving_avg": 1445.857142857143
                                },
                                "upload_time_in_millis": {
                                    "moving_avg": 3964.0
                                }
                            },
                            "download": {}
                        }
                    }
                ]
            }
        }
    }
}
```
</delect>

### 响应字段

远程存储统计API的响应主体分为三类：

*`routing` ：包含与碎片路由有关的信息
*`segment` ：包含与远程段传输有关的统计信息-支持存储
*`translog` ：包含与远程转移转移有关的统计数据-支持存储

#### 路由

这`routing` 对象包含以下字段。

|场地|描述|
|：---|：---|
| `primary` | 表示碎片副本是否是主要碎片。|
| `node` | 分配碎片的节点的名称。|

#### 部分

这`segment.upload` 对象包含以下字段。

|场地|描述|
|：---|：---|
| `local_refresh_timestamp_in_millis` | 最后一个成功的本地刷新时间戳，以毫秒为单位。|
| `remote_refresh_timestamp_in_millis` | 最后一个成功的远程刷新时间戳，以毫秒为单位。|
| `refresh_time_lag_in_millis` | 远程刷新在本地刷新后面的时间，以毫秒为单位。|
| `refresh_lag` | 远程商店落后于本地商店的刷新数量。|
| `bytes_lag` | 远程和本地商店之间的滞后字节。|
| `backpressure_rejection_count` | 由于远程商店中的背压而发出的写拒绝的总数。|
| `consecutive_failure_count` | 自上次成功刷新以来，连续的远程刷新失败的数量。|
| `total_remote_refresh` | 远程刷新的总数。|
| `total_uploads_in_bytes` | 所有上传到远程存储的字节总数。|
| `remote_refresh_size_in_bytes.last_successful` | 在上次成功刷新期间上传的数据大小。|
| `remote_refresh_size_in_bytes.moving_avg` | 数据的平均大小，以字节为单位，在最后一个 * n *刷新中上传。* n*在`remote_store.moving_average_window_size` 环境。有关更多信息，请参阅[远程段背压](https://opensearch.org/docs/latest/tuning-your-cluster/availability-and-recovery/remote-store/remote-segment-backpressure/)。|
| `upload_latency_in_bytes_per_sec.moving_avg` | 最后 * n *上传的远程段上传的平均速度上传每秒。* n*在`remote_store.moving_average_window_size` 环境。有关更多信息，请参阅[远程段背压](https://opensearch.org/docs/latest/tuning-your-cluster/availability-and-recovery/remote-store/remote-segment-backpressure/)。|
| `remote_refresh_latency_in_millis.moving_avg` | 在最后一个 * n *远程刷新期间，单个远程刷新花费的平均时间为毫秒。* n*在`remote_store.moving_average_window_size` 环境。有关更多信息，请参阅[远程段背压](https://opensearch.org/docs/latest/tuning-your-cluster/availability-and-recovery/remote-store/remote-segment-backpressure/)。|

这`segment.download` 对象包含以下字段。

|场地|描述|
|：---|：---|
| `last_sync_timestamp`| 时间戳记，以毫秒为单位，因为最后一个成功的段文件从远程下载-支持存储。|
| `total_download_size.started_bytes` | 从远程积极下载的细分文件的字节总数-支持存储。|
| `total_download_size.succeeded_bytes` | 从远程下载的细分文件的字节总数-支持存储。|
| `total_download_size.failed_bytes` | 无法从远程下载的细分文件的字节总数-后存储。|
| `download_size_in_bytes.last_successful` | 从远程成功下载的最后一个段文件的大小，字节中的大小-支持存储。|
| `download_size_in_bytes.moving_avg`  | 在过去20个下载中下载了段数据的平均段数据大小。|
| `download_speed_in_bytes_per_sec.moving_avg` | 过去20个下载中的平均下载速度，每秒字节。|

#### 翻译

这`translog.upload` 对象包含以下字段。

|场地|描述|
|：---|：---|
| `last_successful_upload_timestamp`| 时间戳记，以毫秒为单位，因为最后一个转换文件成功上传到远程-支持存储。|
| `total_uploads.started` | 尝试翻译的总数上传同步到遥控-支持存储。|
| `total_uploads.failed` | 失败的Translog上传同步到遥控器的总数-支持存储。|
| `total_uploads.succeeded` | 成功的Translog上传同步到遥控器的总数-支持存储。|
| `total_upload_size.started_bytes` | Translog文件的字节总数积极从远程下载-支持存储。|
| `total_upload_size.succeeded_bytes` | Translog文件的字节总数成功上传到远程-支持存储。|
|`total_upload_size.failed_bytes` | 未能上传到远程的Translog文件的字节总数-支持存储。|
| `total_upload_time_in_millis` | 在毫秒内花费的总时间将Translog文件上传到远程-支持存储。|
| `upload_size_in_bytes.moving_avg` | Translog数据的平均大小（以字节为单位）上传到最后一个 * n *下载。* n*在`remote_store.moving_average_window_size` 环境。|
| `upload_speed_in_bytes_per_sec.moving_avg` | 最后 * n *上传的平均翻译上传速度，以每秒字节为单位。* n*在`remote_store.moving_average_window_size` 环境。|
| `upload_time_in_millis.moving_avg` | 自上次 * n *上传以来，单个翻译库上传的平均时间上传。* n*在`remote_store.moving_average_window_size` 环境。|

这`translog.download` 对象包含以下字段。

|场地|描述|
|：---|：---|
| `last_successful_download_timestamp` | 时间戳记，以毫秒为单位，因为最后一个转换文件成功上传到远程-支持存储。|
| `total_downloads.succeeded` | 从远程下载同步的成功转换总数-支持存储。|
| `total_download_size.succeeded_bytes` | Translog文件的字节总数成功地从远程上传-支持存储。|
| `total_download_time_in_millis` | 在毫秒内花费的总时间，从远程下载Translog文件-支持存储。|
| `download_size_in_bytes.moving_avg`  | 在最后一个 * n *下载中下载了Translog数据的平均大小。* n*在`remote_store.moving_average_window_size` 环境。|
| `download_speed_in_bytes_per_sec.moving_avg` | 最后 * n *上传的平均转换下载速度（以字节为单位）。* n*在`remote_store.moving_average_window_size` 环境。|
| `download_time_in_millis.moving_avg` |  自上次 * n *上传以来，单个翻译下载以毫秒下载的平均时间。* n*在`remote_store.moving_average_window_size` 环境。|

## 一个单碎的远程商店统计数据

使用以下API获取单个碎片的远程存储统计信息。

#### 示例请求

```json
GET _remotestore/stats/<index_name>/<shard_id>
```
{％包含副本-curl.html％}

#### 示例响应

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-delta}

```json
{
    "_shards": {
        "total": 2,
        "successful": 2,
        "failed": 0
    },
    "indices": {
        "remote-index": {
            "shards": {
                "0": [
                    {
                        "routing": {
                            "state": "STARTED",
                            "primary": true,
                            "node": "q1VxWZnCTICrfRc2bRW3nw"
                        },
                        "segment": {
                            "download": {},
                            "upload": {
                                "local_refresh_timestamp_in_millis": 1694171634102,
                                "remote_refresh_timestamp_in_millis": 1694171634102,
                                "refresh_time_lag_in_millis": 0,
                                "refresh_lag": 0,
                                "bytes_lag": 0,
                                "backpressure_rejection_count": 0,
                                "consecutive_failure_count": 0,
                                "total_uploads": {
                                    "started": 5,
                                    "succeeded": 5,
                                    "failed": 0
                                },
                                "total_upload_size": {
                                    "started_bytes": 15342,
                                    "succeeded_bytes": 15342,
                                    "failed_bytes": 0
                                },
                                "remote_refresh_size_in_bytes": {
                                    "last_successful": 0,
                                    "moving_avg": 3068.4
                                },
                                "upload_speed_in_bytes_per_sec": {
                                    "moving_avg": 99988.2
                                },
                                "remote_refresh_latency_in_millis": {
                                    "moving_avg": 44.0
                                }
                            }
                        },
                        "translog": {
                            "upload": {
                                "last_successful_upload_timestamp": 1694171633644,
                                "total_uploads": {
                                    "started": 6,
                                    "failed": 0,
                                    "succeeded": 6
                                },
                                "total_upload_size": {
                                    "started_bytes": 1932,
                                    "failed_bytes": 0,
                                    "succeeded_bytes": 1932
                                },
                                "total_upload_time_in_millis": 21478,
                                "upload_size_in_bytes": {
                                    "moving_avg": 322.0
                                },
                                "upload_speed_in_bytes_per_sec": {
                                    "moving_avg": 2073.8333333333335
                                },
                                "upload_time_in_millis": {
                                    "moving_avg": 3579.6666666666665
                                }
                            },
                            "download": {}
                        }
                    },
                    {
                        "routing": {
                            "state": "STARTED",
                            "primary": false,
                            "node": "EZuen5Y5Sv-eDCLwh9gv-Q"
                        },
                        "segment": {
                            "download": {
                                "last_sync_timestamp": 1694171634148,
                                "total_download_size": {
                                    "started_bytes": 15112,
                                    "succeeded_bytes": 15112,
                                    "failed_bytes": 0
                                },
                                "download_size_in_bytes": {
                                    "last_successful": 2910,
                                    "moving_avg": 1259.3333333333333
                                },
                                "download_speed_in_bytes_per_sec": {
                                    "moving_avg": 382387.3333333333
                                }
                            },
                            "upload": {}
                        },
                        "translog": {
                            "upload": {},
                            "download": {}
                        }
                    }
                ]
            }
        }
    }
}
```
</delect>

### 本地碎片的远程商店统计数据

如果您只想在服务远程存储统计API请求的节点上仅获取碎片，请设置`local` 查询参数为`true`，如以下示例请求所示：


```json
GET _remotestore/stats/<index_name>?local=true
```
{％包含副本-curl.html％}


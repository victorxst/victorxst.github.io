---
layout: default
title: 获取快照状态
parent: 快照API
nav_order: 8
---

# 获取快照状态
**引入1.0**
{: .label .label-purple }

在快照创建期间和之后返回有关快照状态的详细信息。

要了解快照创作，请参阅[创建快照]({{site.url}}{{site.baseurl}}/api-reference/snapshots/create-snapshot)。

如果使用安全插件，则必须`monitor_snapshot`，`create_snapshot`， 或者`manage cluster` 特权。
{: .note}

## 路径参数

路径参数是可选的。

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 存储库| 细绳| 包含快照的存储库。|
| 快照| 细绳| 快照要返回。|

三个请求变体提供了灵活性：

*`GET _snapshot/_status` 返回所有存储库中当前运行快照的状态。

*`GET _snapshot/<repository>/_status` 返回仅在指定存储库中运行快照的状态。这是首选变体。

*`GET _snapshot/<repository>/<snapshot>/_status` 返回指定存储库中所有快照的状态，无论它们是否正在运行。

除了当前运行快照外，使用API以返回状态，对于（1）机器资源，如果在云中运行，则可以非常昂贵。对于每个快照，每个请求都会导致文件从所有快照的碎片中读取。
{: .warning}

## 请求字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| ignore_unavailable| 布尔| 如何处理不可用快照的请求。如果`false`，该请求返回无法使用快照的错误。如果`true`，该请求忽略了不可用的快照，例如被损坏或暂时返回的快照。默认为`false`。|

#### 示例请求

以下请求返回`my-first-snapshot` 在里面`my-opensearch-repo` 存储库。不可用的快照被忽略。

````json
GET _snapshot/my-opensearch-repo/my-first-snapshot/_status
{
   "ignore_unavailable": true 
}
````
{% include copy-curl.html %}

#### 示例响应

接下来的示例对应于上面的请求[示例请求](#example-request) 部分。

这`GET _snapshot/my-opensearch-repo/my-first-snapshot/_status` 请求返回以下字段：

````json
{
  "snapshots" : [
    {
      "snapshot" : "my-first-snapshot",
      "repository" : "my-opensearch-repo",
      "uuid" : "dCK4Qth-TymRQ7Tu7Iga0g",
      "state" : "SUCCESS",
      "include_global_state" : true,
      "shards_stats" : {
        "initializing" : 0,
        "started" : 0,
        "finalizing" : 0,
        "done" : 7,
        "failed" : 0,
        "total" : 7
      },
      "stats" : {
        "incremental" : {
          "file_count" : 31,
          "size_in_bytes" : 24488927
        },
        "total" : {
          "file_count" : 31,
          "size_in_bytes" : 24488927
        },
        "start_time_in_millis" : 1660666841667,
        "time_in_millis" : 14054
      },
      "indices" : {
        ".opensearch-observability" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "total" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "start_time_in_millis" : 1660666841868,
            "time_in_millis" : 201
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "total" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "start_time_in_millis" : 1660666841868,
                "time_in_millis" : 201
              }
            }
          }
        },
        "shakespeare" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 4,
              "size_in_bytes" : 18310117
            },
            "total" : {
              "file_count" : 4,
              "size_in_bytes" : 18310117
            },
            "start_time_in_millis" : 1660666842470,
            "time_in_millis" : 13050
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 4,
                  "size_in_bytes" : 18310117
                },
                "total" : {
                  "file_count" : 4,
                  "size_in_bytes" : 18310117
                },
                "start_time_in_millis" : 1660666842470,
                "time_in_millis" : 13050
              }
            }
          }
        },
        "opensearch_dashboards_sample_data_flights" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 10,
              "size_in_bytes" : 6132245
            },
            "total" : {
              "file_count" : 10,
              "size_in_bytes" : 6132245
            },
            "start_time_in_millis" : 1660666843476,
            "time_in_millis" : 6221
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 10,
                  "size_in_bytes" : 6132245
                },
                "total" : {
                  "file_count" : 10,
                  "size_in_bytes" : 6132245
                },
                "start_time_in_millis" : 1660666843476,
                "time_in_millis" : 6221
              }
            }
          }
        },
        ".opendistro-reports-definitions" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "total" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "start_time_in_millis" : 1660666843076,
            "time_in_millis" : 200
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "total" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "start_time_in_millis" : 1660666843076,
                "time_in_millis" : 200
              }
            }
          }
        },
        ".opendistro-reports-instances" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "total" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "start_time_in_millis" : 1660666841667,
            "time_in_millis" : 201
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "total" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "start_time_in_millis" : 1660666841667,
                "time_in_millis" : 201
              }
            }
          }
        },
        ".kibana_1" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 13,
              "size_in_bytes" : 45733
            },
            "total" : {
              "file_count" : 13,
              "size_in_bytes" : 45733
            },
            "start_time_in_millis" : 1660666842673,
            "time_in_millis" : 2007
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 13,
                  "size_in_bytes" : 45733
                },
                "total" : {
                  "file_count" : 13,
                  "size_in_bytes" : 45733
                },
                "start_time_in_millis" : 1660666842673,
                "time_in_millis" : 2007
              }
            }
          }
        },
        ".opensearch-notifications-config" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "total" : {
              "file_count" : 1,
              "size_in_bytes" : 208
            },
            "start_time_in_millis" : 1660666842270,
            "time_in_millis" : 200
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "total" : {
                  "file_count" : 1,
                  "size_in_bytes" : 208
                },
                "start_time_in_millis" : 1660666842270,
                "time_in_millis" : 200
              }
            }
          }
        }
      }
    }
  ]
}
````

## 响应字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 存储库| 细绳| 包含快照的存储库的名称。|
| 快照| 细绳| 快照名称。|
| UUID| 细绳| 快照普遍唯一的标识符（UUID）。|
| 状态| 细绳| 快照的当前状态。看[快照状态](#snapshot-states)。|
| 包括_global_state| 布尔| 快照中是否包含当前的群集状态。|
| shards_stats| 目的| 快照的碎片计数。看[碎片统计](#shard-stats)。|
| 统计| 目的| 快照中包含的文件的详细信息。`file_count`：文件数。`size_in_bytes`：总体大小。看[快照文件统计](#snapshot-file-stats)。|
| 指数| 对象列表| 包含有关快照中索引的信息的对象列表。看[索引对象](#index-objects)。|

##### 快照状态

| 状态| 描述| 
:--- | :--- |
| 失败的| 快照以错误终止，没有存储数据。|
| 进行中| 快照当前正在运行。|
| 部分的| 存储了全球群集状态，但没有存储来自至少一个碎片的数据。这`failures` 属性[创建快照]({{site.url}}{{site.baseurl}}/api-reference/snapshots/create-snapshot) 响应包含其他细节。|
| 成功| 快照完成，所有碎片都成功地存储了。|

##### 碎片统计

所有属性值都是整数。

| 财产| 描述| 
:--- | :--- |
| 初始化| 仍在初始化的碎片数量。|
| 开始| 已经开始但未完成的碎片数量。|
| 最终确定| 最终确定但未完成的碎片数量。|
| 完毕| 成功初始化，启动和最终确定的碎片数量。|
| 失败的| 未能包含在快照中的碎片数。|
| 全部的| 快照中包含的碎片总数。|

##### 快照文件统计

| 财产| 类型| 描述| 
:--- | :--- | :--- |
| 增加的| 目的| 快照创建期间仍需要复制文件的数量和大小。对于完整的快照，`incremental` 提供存储库中尚未在存储库中的文件的数量和大小，并作为增量快照的一部分复制。|
| 处理| 目的| 已经上传到快照的文件的数字和大小。处理过`file_count` 和`size_in_bytes` 在上传文件后，将在统计数据中增加。|
| 全部的| 目的| 快照引用的文件的总数和大小。| 
| start_time_in_millis| 长的| 时间（以毫秒为单位）开始创建快照。|
| time_in_millis| 长的| 快照完成的总时间（以毫秒为单位）。|

##### 索引对象

| 财产| 类型| 描述| 
:--- | :--- | :--- |
| shards_stats| 目的| 看[碎片统计](#shard-stats)。|
| 统计| 目的| 看[快照文件统计](#snapshot-file-stats)。|
| 碎片| 对象列表| 包含有关碎片信息的对象列表，其中包括快照。碎片的工作人员在下面列出了大胆的文本。<br /> <br />**阶段**：快照中的当前状态。碎片状态为：<br /> <br /> *完成：快照中成功存储在存储库中的碎片数。<br /> <br /> *故障：快照中未成功存储在存储库中的碎片数。<br /> <br /> *最终确定：在存储在存储库中的最终确定阶段中的碎片数。<br /> <br />* init：快照中存储在存储库中的初始化阶段中的碎片数。<br /> <br />*开始：快照中的碎片数存储在存储库中的开始阶段。<br /> <br />**统计**： 看[快照文件统计](#snapshot-file-stats)。<br /> <br />**全部的**：快照引用的文件的总数和大小。<br /> <br />**start_time_in_millis**：时间（以毫秒为单位）开始快照创建。<br /> <br />**time_in_millis**：快照完成的总时间（以毫秒为单位）。


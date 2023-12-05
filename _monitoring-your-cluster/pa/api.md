---
layout: default
title: API
parent: 性能分析仪
nav_order: 1
redirect_from:
  - /monitoring-plugins/pa/api/
---

# 性能分析仪API
引入1.0
{: .label .label-purple }

性能分析仪用于大多数请求使用单个HTTP方法和URI：

```
GET <endpoint>:9600/_plugins/_performanceanalyzer/metrics
```

请注意使用端口9600。提供指标，聚合，尺寸和节点（可选）的参数：

```
?metrics=<metrics>&agg=<aggregations>&dim=<dimensions>&nodes=all"
```

有关指标的完整列表，请参阅[指标参考]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/reference/)。性能分析仪每五秒钟一次更新其数据。如果创建自定义客户端，我们建议使用相同的间隔来调用API。


#### 示例请求

```
GET localhost:9600/_plugins/_performanceanalyzer/metrics?metrics=Latency,CPU_Utilization&agg=avg,max&dim=ShardID&nodes=all
```


#### 示例响应

```json
{
  "keHlhQbbTpm1BYicficEQg": {
    "timestamp": 1554940530000,
    "data": {
      "fields": [{
          "name": "ShardID",
          "type": "VARCHAR"
        },
        {
          "name": "Latency",
          "type": "DOUBLE"
        },
        {
          "name": "CPU_Utilization",
          "type": "DOUBLE"
        }
      ],
      "records": [
        [
          null,
          null,
          0.012552206029147535
        ],
        [
          "1",
          4.8,
          0.0009780939762972104
        ]
      ]
    }
  },
  "bHdpbMJZTs-TKtZro2SmYA": {
    "timestamp": 1554940530000,
    "data": {
      "fields": [{
          "name": "ShardID",
          "type": "VARCHAR"
        },
        {
          "name": "Latency",
          "type": "DOUBLE"
        },
        {
          "name": "CPU_Utilization",
          "type": "DOUBLE"
        }
      ],
      "records": [
        [
          null,
          18.2,
          0.011966493817311527
        ],
        [
          "1",
          14.8,
          0.0007670829370071493
        ]
      ]
    }
  }
}
```

在这种情况下，每个顶部-级别对象代表一个节点。API返回您指定的指标和尺寸的名称和数据类型，以及五秒钟前的值以及当前值（如果不同的话）。零值表示在此期间的无活动。

性能分析仪还有一个额外的URI，可以返回每个度量标准的设备。


#### 示例请求

```
GET localhost:9600/_plugins/_performanceanalyzer/metrics/units
```


#### 示例响应

```json
{
  "Disk_Utilization": "%",
  "Cache_Request_Hit": "count",
  "HTTP_RequestDocs": "count",
  "Net_TCP_Lost": "segments/flow",
  "Refresh_Time": "ms",
  "GC_Collection_Event": "count",
  "Merge_Time": "ms",
  "Sched_CtxRate": "count/s",
  "Cache_Request_Size": "B",
  "ThreadPool_QueueSize": "count",
  "Sched_Runtime": "s/ctxswitch",
  "Disk_ServiceRate": "MB/s",
  "Heap_AllocRate": "B/s",
  "Heap_Max": "B",
  "Sched_Waittime": "s/ctxswitch",
  "ShardBulkDocs": "count",
  "Thread_Blocked_Time": "s/event",
  "VersionMap_Memory": "B",
  "Master_Task_Queue_Time": "ms",
  "Merge_CurrentEvent": "count",
  "Indexing_Buffer": "B",
  "Bitset_Memory": "B",
  "Net_PacketDropRate4": "packets/s",
  "Heap_Committed": "B",
  "Net_PacketDropRate6": "packets/s",
  "Thread_Blocked_Event": "count",
  "GC_Collection_Time": "ms",
  "Cache_Query_Miss": "count",
  "IO_TotThroughput": "B/s",
  "Latency": "ms",
  "Net_PacketRate6": "packets/s",
  "Cache_Query_Hit": "count",
  "IO_ReadSyscallRate": "count/s",
  "Net_PacketRate4": "packets/s",
  "Cache_Request_Miss": "count",
  "CB_ConfiguredSize": "B",
  "CB_TrippedEvents": "count",
  "ThreadPool_RejectedReqs": "count",
  "Disk_WaitTime": "ms",
  "Net_TCP_TxQ": "segments/flow",
  "Master_Task_Run_Time": "ms",
  "IO_WriteSyscallRate": "count/s",
  "IO_WriteThroughput": "B/s",
  "Flush_Event": "count",
  "Net_TCP_RxQ": "segments/flow",
  "Refresh_Event": "count",
  "Flush_Time": "ms",
  "Heap_Init": "B",
  "CPU_Utilization": "cores",
  "HTTP_TotalRequests": "count",
  "ThreadPool_ActiveThreads": "count",
  "Cache_Query_Size": "B",
  "Paging_MinfltRate": "count/s",
  "Merge_Event": "count",
  "Net_TCP_SendCWND": "B/flow",
  "Cache_Request_Eviction": "count",
  "Segments_Total": "count",
  "Heap_Used": "B",
  "Cache_FieldData_Eviction": "count",
  "IO_TotalSyscallRate": "count/s",
  "CB_EstimatedSize": "B",
  "Net_Throughput": "B/s",
  "Paging_RSS": "pages",
  "Indexing_ThrottleTime": "ms",
  "IndexWriter_Memory": "B",
  "Master_PendingQueueSize": "count",
  "Net_TCP_SSThresh": "B/flow",
  "Cache_FieldData_Size": "B",
  "Paging_MajfltRate": "count/s",
  "ThreadPool_TotalThreads": "count",
  "IO_ReadThroughput": "B/s",
  "ShardEvents": "count",
  "Net_TCP_NumFlows": "count"
}
```


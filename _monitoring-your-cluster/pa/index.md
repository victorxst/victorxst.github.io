---
layout: default
title: 性能分析仪
nav_order: 58
has_children: true
redirect_from:
  - /monitoring-plugins/pa/
  - /monitoring-plugins/pa/index/
---

# 性能分析仪

Performance Analyzer是一个包含代理和REST API的插件，可让您查询众多集群性能指标，包括这些指标的聚合。

默认情况下，在OpenSearch版本2.0及更高版本中安装了性能分析仪插件。如果要使用禁用性能分析仪使用OpenSearch 2.0或以后，请参见[禁用性能分析仪](#disable-performance-analyzer)。
{: .note}

## 先决条件

在使用OpenSearch使用性能分析仪之前，请查看以下先决条件。

### 贮存

性能分析仪使用`/dev/shm` 用于临时存储。在重型集群工作负载期间，性能分析仪最多可以使用1 GB的空间。

但是，Docker有一个默认`/dev/shm` 大小为64 MB。要更改此值，您可以使用`docker run --shm-size 1gb` 标志或[Docker中的类似设置](https://docs.docker.com/compose/compose-file#shm_size)。

如果您不使用Docker，则可以检查`/dev/shm` 使用`df -h`。默认值应该足够，但是如果您需要更改其大小，将以下行添加到`/etc/fstab`：

```bash
tmpfs /dev/shm tmpfs defaults,noexec,nosuid,size=1G 0 0
```

然后重新安装文件系统：

```bash
mount -o remount /dev/shm
```

### 安全

绩效分析仪支持请求中的运输中的加密。当前，它 *不 *支持请求客户端或服务器身份验证。要在运输中启用加密，请编辑`performance-analyzer.properties` 在你的`$OPENSEARCH_HOME` 目录：

```properties
vi $OPENSEARCH_HOME/config/opensearch-performance-analyzer/performance-analyzer.properties
```

更改以下行以在运输中配置加密。注意`certificate-file-path` 必须是服务器的证书，而不是根证书授权（CA）。

````properties
https-enabled = true

#Setup the correct path for certificates
certificate-file-path = specify_path

private-key-file-path = specify_path
````

## 安装性能分析仪

性能分析仪插件包含在安装中[Docker]({{site.url}}{{site.baseurl}}/opensearch/install/docker/) 和[tarball]({{site.url}}{{site.baseurl}}/opensearch/install/tar/)，但是您也可以手动安装插件。

要手动安装性能分析仪插件，请从[小牛](https://search.maven.org/search?q=org.opensearch.plugin) 并使用标准安装[插件安装]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/) 过程。性能分析仪在集群中的每个节点上运行。

要在Tarball安装上启动Performance Analyzer根本原因分析（RCA）代理，请运行以下命令：
      
````bash
OPENSEARCH_HOME=~/opensearch-2.2.1 OPENSEARCH_JAVA_HOME=~/opensearch-2.2.1/jdk OPENSEARCH_PATH_CONF=~/opensearch-2.2.1/bin ./performance-analyzer-agent-cli
````

以下命令启用性能分析仪插件。

````bash
curl -XPOST localhost:9200/_plugins/_performanceanalyzer/cluster/config -H 'Content-Type: application/json' -d '{"enabled": true}'
````

## 禁用性能分析仪

如果您希望通过禁用Performance Analyzer插件来保存内存并运行OpenSearch的本地实例，请执行以下步骤：

1. 在禁用性能分析仪之前，请使用以下命令停止任何当前运行的RCA代理操作：

  ```bash
  curl -XPOST localhost:9200/_plugins/_performanceanalyzer/rca/cluster/config -H 'Content-Type: application/json' -d '{"enabled": false}'
  ```

2. 通过运行以下命令来关闭性能分析仪RCA代理：

  ```bash
  kill $(ps aux | grep -i 'PerformanceAnalyzerApp' | grep -v grep | awk '{print $2}')
  ```

3. 通过运行以下命令来禁用性能分析仪插件：

  ```bash
  curl -XPOST localhost:9200/_plugins/_performanceanalyzer/cluster/config -H 'Content-Type: application/json' -d '{"enabled": false}'
  ```

4. 通过运行以下命令来卸载性能分析仪插件：

  ```bash
  bin/opensearch-plugin remove opensearch-performance-analyzer
  ```

## 配置性能分析仪

要配置性能分析仪插件，请编辑`performance-analyzer.properties` 配置文件`config/opensearch-performance-analyzer/` 目录。确保不加句子`#webservice-bind-host` 并将其设置为`0.0.0.0`。您可以参考以下示例配置。

````bash
# ======================== OpenSearch Performance Analyzer plugin config =========================

# NOTE: this is an example for Linux. Please modify the config accordingly if you are using it under other OS.

# WebService bind host; default to all interfaces
webservice-bind-host = 0.0.0.0

# Metrics data location
metrics-location = /dev/shm/performanceanalyzer/

# Metrics deletion interval (minutes) for metrics data.
# Interval should be between 1 to 60.
metrics-deletion-interval = 1

# If set to true, the system cleans up the files behind it. So at any point, we should expect only 2
# metrics-db-file-prefix-path files. If set to false, no files are cleaned up. This can be useful, if you are archiving
# the files and wouldn't like for them to be cleaned up.
cleanup-metrics-db-files = true

# WebService exposed by App's port
webservice-listener-port = 9600

# Metric DB File Prefix Path location
metrics-db-file-prefix-path = /tmp/metricsdb_

https-enabled = false

#Setup the correct path for certificates
#certificate-file-path = specify_path

#private-key-file-path = specify_path

# Plugin Stats Metadata file name, expected to be in the same location
plugin-stats-metadata = plugin-stats-metadata

# Agent Stats Metadata file name, expected to be in the same location
agent-stats-metadata = agent-stats-metadata
````
要启动性能分析仪RCA代理，请运行以下命令：

````bash
OPENSEARCH_HOME=~/opensearch-2.2.1 OPENSEARCH_JAVA_HOME=~/opensearch-2.2.1/jdk OPENSEARCH_PATH_CONF=~/opensearch-2.2.1/bin ./performance-analyzer-agent-cli
````


## 启用rpm/yum安装的性能分析仪

如果您从RPM分发安装了OpenSearch，则可以使用`systemctl`：

```bash
# Start OpenSearch Performance Analyzer
sudo systemctl start opensearch-performance-analyzer.service
# Stop OpenSearch Performance Analyzer
sudo systemctl stop opensearch-performance-analyzer.service
```

## 示例API查询和响应

以下是示例性能分析仪API查询。查询拉出与您的OpenSearch集群相关的性能指标：
  
````bash
GET localhost:9600/_plugins/_performanceanalyzer/metrics/units
````

以下是一个示例响应：

````json
{"Disk_Utilization":"%","Cache_Request_Hit":"count", 
"Refresh_Time":"ms","ThreadPool_QueueLatency":"count",
"Merge_Time":"ms","ClusterApplierService_Latency":"ms",
"PublishClusterState_Latency":"ms",
"Cache_Request_Size":"B","LeaderCheck_Failure":"count",
"ThreadPool_QueueSize":"count","Sched_Runtime":"s/ctxswitch","Disk_ServiceRate":"MB/s","Heap_AllocRate":"B/s","Indexing_Pressure_Current_Limits":"B",
"Sched_Waittime":"s/ctxswitch","ShardBulkDocs":"count",
"Thread_Blocked_Time":"s/event","VersionMap_Memory":"B",
"Master_Task_Queue_Time":"ms","IO_TotThroughput":"B/s",
"Indexing_Pressure_Current_Bytes":"B",
"Indexing_Pressure_Last_Successful_Timestamp":"ms",
"Net_PacketRate6":"packets/s","Cache_Query_Hit":"count",
"IO_ReadSyscallRate":"count/s","Net_PacketRate4":"packets/s","Cache_Request_Miss":"count",
"ThreadPool_RejectedReqs":"count","Net_TCP_TxQ":"segments/flow","Master_Task_Run_Time":"ms",
"IO_WriteSyscallRate":"count/s","IO_WriteThroughput":"B/s",
"Refresh_Event":"count","Flush_Time":"ms","Heap_Init":"B",
"Indexing_Pressure_Rejection_Count":"count",
"CPU_Utilization":"cores","Cache_Query_Size":"B",
"Merge_Event":"count","Cache_FieldData_Eviction":"count",
"IO_TotalSyscallRate":"count/s","Net_Throughput":"B/s",
"Paging_RSS":"pages",
"AdmissionControl_ThresholdValue":"count",
"Indexing_Pressure_Average_Window_Throughput":"count/s",
"Cache_MaxSize":"B","IndexWriter_Memory":"B",
"Net_TCP_SSThresh":"B/flow","IO_ReadThroughput":"B/s",
"LeaderCheck_Latency":"ms","FollowerCheck_Failure":"count",
"HTTP_RequestDocs":"count","Net_TCP_Lost":"segments/flow",
"GC_Collection_Event":"count","Sched_CtxRate":"count/s",
"AdmissionControl_RejectionCount":"count","Heap_Max":"B",
"ClusterApplierService_Failure":"count",
"PublishClusterState_Failure":"count",
"Merge_CurrentEvent":"count","Indexing_Buffer":"B",
"Bitset_Memory":"B","Net_PacketDropRate4":"packets/s",
"Heap_Committed":"B","Net_PacketDropRate6":"packets/s",
"Thread_Blocked_Event":"count","GC_Collection_Time":"ms",
"Cache_Query_Miss":"count","Latency":"ms",
"Shard_State":"count","Thread_Waited_Event":"count",
"CB_ConfiguredSize":"B","ThreadPool_QueueCapacity":"count",
"CB_TrippedEvents":"count","Disk_WaitTime":"ms",
"Data_RetryingPendingTasksCount":"count",
"AdmissionControl_CurrentValue":"count",
"Flush_Event":"count","Net_TCP_RxQ":"segments/flow",
"Shard_Size_In_Bytes":"B","Thread_Waited_Time":"s/event",
"HTTP_TotalRequests":"count",
"ThreadPool_ActiveThreads":"count",
"Paging_MinfltRate":"count/s","Net_TCP_SendCWND":"B/flow",
"Cache_Request_Eviction":"count","Segments_Total":"count",
"FollowerCheck_Latency":"ms","Heap_Used":"B",
"Master_ThrottledPendingTasksCount":"count",
"CB_EstimatedSize":"B","Indexing_ThrottleTime":"ms",
"Master_PendingQueueSize":"count",
"Cache_FieldData_Size":"B","Paging_MajfltRate":"count/s",
"ThreadPool_TotalThreads":"count","ShardEvents":"count",
"Net_TCP_NumFlows":"count","Election_Term":"count"}
````

## 根本原因分析

这[根本原因分析]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/rca/index/) （RCA）框架使用绩效分析仪的信息将其群集遇到的性能和可用性问题的根本原因告知管理员。

### 启用RCA框架

要启用RCA框架，请运行以下命令：

```bash
curl -XPOST http://localhost:9200/_plugins/_performanceanalyzer/rca/cluster/config -H 'Content-Type: application/json' -d '{"enabled": true}'
```

如果您遇到`curl: (52) Empty reply from server` 响应，运行以下命令以启用RCA：

```bash
curl -XPOST https://localhost:9200/_plugins/_performanceanalyzer/rca/cluster/config -H 'Content-Type: application/json' -d '{"enabled": true}' -u 'admin:admin' -k
```

### 示例API查询和响应

要请求所有可用的RCA，请运行以下命令：

````bash
GET localhost:9600/_plugins/_performanceanalyzer/rca
````

要请求特定的RCA，请运行以下命令：

````bash
GET localhost:9600/_plugins/_performanceanalyzer/rca?name=HighHeapUsageClusterRCA
````

以下是一个示例响应：

```json
{
  "HighHeapUsageClusterRCA": [{
    "RCA_name": "HighHeapUsageClusterRCA",
    "state": "unhealthy",
    "timestamp": 1587426650942,
    "HotClusterSummary": [{
      "number_of_nodes": 2,
      "number_of_unhealthy_nodes": 1,
      "HotNodeSummary": [{
        "host_address": "192.168.144.2",
        "node_id": "JtlEoRowSI6iNpzpjlbp_Q",
        "HotResourceSummary": [{
          "resource_type": "old gen",
          "threshold": 0.65,
          "value": 0.81827232588145373,
          "avg": NaN,
          "max": NaN,
          "min": NaN,
          "unit_type": "heap usage in percentage",
          "time_period_seconds": 600,
          "TopConsumerSummary": [{
              "name": "CACHE_FIELDDATA_SIZE",
              "value": 590702564
            },
            {
              "name": "CACHE_REQUEST_SIZE",
              "value": 28375
            },
            {
              "name": "CACHE_QUERY_SIZE",
              "value": 12687
            }
          ],
        }]
      }]
    }]
  }]
}
```


### 相关链接

可以在以下链接中找到有关性能分析仪和RCA使用的进一步文档：

- [性能分析仪API]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/pa/api/)
- [根本原因分析]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/pa/rca/index/)
- [根本原因分析]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/pa/rca/api/)。
- [RFC：根本原因分析](https://github.com/opensearch-project/performance-analyzer-rca/blob/main/docs/rfc-rca.pdf)


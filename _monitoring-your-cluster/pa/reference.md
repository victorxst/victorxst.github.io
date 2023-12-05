---
layout: default
title: 指标参考
parent: 性能分析仪
nav_order: 3
redirect_from:
  - /monitoring-plugins/pa/reference/
---

# 指标参考

性能分析仪提供了许多指标来帮助您评估性能。以下表描述了可用的指标，该指标由与该指标最相关的尺寸分组。所有指标都支持`avg`，`sum`，`min`， 和`max` 汇总，尽管对于某些指标，但无论聚集类型如何，测量值都是相同的。

有关每个维度的信息，请参见[维度参考](#dimensions-reference) 在此主题的后面。

此列表是广泛的。我们建议使用CTRL/CMD + F查找所需的内容。
{: .tip }

## 相关维度：`ShardID`，`IndexName`，`Operation`，`ShardRole`

<table>
 <thead样式="text-align: left">
    <tr>
      <th>公制</th>
      <th>描述</th>
    </tr>
 </thead>
  <tbody>
    <tr>
      <td> cpu_utilization
      </td>
      <td> CPU使用率。在过去五秒钟内，相关线使用的CPU时间（以毫秒为单位）除以5000毫秒。
      </td>
    </tr>
    <tr>
      <td> paging_majfltrate
      </td>
      <td>在过去五秒钟内每秒的主要故障数量。一个主要故障需要该过程从磁盘加载存储页面。
      </td>
    </tr>
    <tr>
      <td> paging_minfltrate
      </td>
      <td>在过去五秒钟内每秒的小故障数量。次要故障不需要从磁盘加载内存页面的过程。
      </td>
    </tr>
    <tr>
      <td> paging_rss
      </td>
      <td>该过程在真实内存中具有的页数---计算文本，数据或堆栈空间的页面。这个数字不包括尚未要求的页面-加载或交换。
      </td>
    </tr>
    <tr>
      <td> sched_runtime
      </td>
      <td>时间（秒）在每个上下文开关上执行CPU。
      </td>
    </tr>
    <tr>
      <td> sched_waittime
      </td>
      <td>时间（秒）在每个上下文开关等待运行队列上花费。
      </td>
    </tr>
    <tr>
      <td> sched_ctxrate
      </td>
      <td>在过去五秒钟内每秒运行的CPU次数。
      </td>
    </tr>
    <tr>
      <td> HEAP_OLLOCRATE
      </td>
      <td>在过去5秒内分配的每秒分配的堆内存的近似值。
      </td>
    </tr>
    <tr>
      <td> io_readThroughtup
      </td>
      <td>在过去五秒钟内每秒读取的字节数。
      </td>
    </tr>
    <tr>
      <td> io_writeThrougt
      </td>
      <td>在过去五秒钟内每秒写的字节数。
      </td>
    </tr>
    <tr>
      <td> io_totThroughtup
      </td>
      <td>在过去五秒钟内每秒读取或书写的字节数。
      </td>
    </tr>
    <tr>
      <td> io_readsyscallrate
      </td>
      <td>在过去五秒钟内每秒读取系统呼叫。
      </td>
    </tr>
    <tr>
      <td> io_writesyscallrate
      </td>
      <td>在过去五秒钟内每秒写系统呼叫。
      </td>
    </tr>
    <tr>
      <td> io_totalsyscallrate
      </td>
      <td>在过去五秒钟内每秒读取和写入系统调用。
      </td>
    </tr>
    <tr>
      <td> thread_blocked_time
      </td>
      <td>平均时间为几秒钟的时间，即关联的线程已被阻止输入或重新输入监视器。
      </td>
    </tr>
    <tr>
      <td> thread_blocked_event
      </td>
      <td>关联线程已阻止输入或重新输入监视器的总数（即，线程已在`blocked` 状态）。
      </td>
    </tr>
    <tr>
      <td> thread_waited_time
      </td>
      <td>相关线程已经等待输入或重新输入监视器的平均时间（即，线程已在`WAITING` 或者`TIMED_WAITING` 状态）”。
      </td>
    </tr>
    <tr>
      <td> thread_waited_event
      </td>
      <td>关联线程已经等待输入或重新输入监视器的总次数（即，在<code>等待</code>或<code> timed_waiting </code </code>>状态）。
      </td>
    </tr>
    <tr>
      <td> shardevents
      </td>
      <td>在过去五秒钟内，在碎片上执行的事件总数。
      </td>
    </tr>
    <tr>
      <td> shardbulkdocs
      </td>
      <td>在过去五秒钟内索引的文档总数。
      </td>
    </tr>
  </tbody>
</table>

## 相关维度：`ShardID`，`IndexName` 

<table>
  <thead样式="text-align: left">
    <tr>
      <th>公制</th>
      <th>描述</th>
    </tr>
  </thead>
  <tbody>
  <tr>
      <td>索引_throttletime
      </td>
      <td>时间（毫秒），该指数在过去五秒钟内一直处于合并节流控制状态。
      </td>
    </tr>
    <tr>
      <td> cache_query_hit
      </td>
      <td>过去五秒钟中查询缓存中成功查找的数量。
      </td>
    </tr>
    <tr>
      <td> cache_query_miss
      </td>
      <td>查询缓存中未能检索的查找数量`DocIdSet` 在过去的五秒钟中。`DocIdSet` 是Lucene中的一组文档ID。
      </td>
    </tr>
    <tr>
      <td> cache_query_size
      </td>
      <td>查询缓存内存大小在字节中。
      </td>
    </tr>
    <tr>
      <td> cache_fielddata_eviction
      </td>
      <td>在过去五秒钟内，OpenSearch的次数已从FieldData Heap空间驱逐数据（发生在堆积空间时）。
      </td>
    </tr>
    <tr>
      <td> cache_fielddata_size
      </td>
      <td>字节中的fieldData内存大小。
      </td>
    </tr>
    <tr>
      <td> cache_request_hit
      </td>
      <td>过去五秒钟内，碎片请求缓存中成功查找的数量。
      </td>
    </tr>
    <tr>
      <td> cache_request_miss
      </td>
      <td>在过去五秒钟内未能检索搜索请求结果的请求缓存中的查找数量。
      </td>
    </tr>
    <tr>
      <td> cache_request_eviction
      </td>
      <td>在过去五秒钟内，OpenSearch的次数OpenSearch驱逐出从Shard Request Cache中驱逐数据（发生在请求缓存已满时）。
      </td>
    </tr>
    <tr>
      <td> cache_request_size
      </td>
      <td> shard请求缓存内存大小在字节中。
      </td>
    </tr>
    <tr>
      <td> refresh_event
      </td>
      <td>在过去五秒钟内执行的刷新总数。
      </td>
    </tr>
    <tr>
      <td> refresh_time
      </td>
      <td>总时间（毫秒）在过去的五秒钟内花费了刷新
      </td>
    </tr>
    <tr>
      <td> flush_event
      </td>
      <td>过去五秒钟内执行的冲洗总数。
      </td>
    </tr>
    <tr>
      <td> flush_time
      </td>
      <td>在过去五秒钟内，总时间（毫秒）花费了冲洗。
      </td>
    </tr>
    <tr>
      <td> Merge_event
      </td>
      <td>在过去五秒钟内执行的合并总数。
      </td>
    </tr>
    <tr>
      <td> Merge_time
      </td>
      <td>在过去五秒钟内，总时间（毫秒）花费了合并。
      </td>
    </tr>
    <tr>
      <td> Merge_currentevent
      </td>
      <td>当前的合并执行数。
      </td>
    </tr>
    <tr>
      <td>索引_Buffer
      </td>
      <td>字节中的索引缓冲区内存大小。
      </td>
    </tr>
    <tr>
      <td> segments_total
      </td>
      <td>段数。
      </td>
    </tr>
    <tr>
      <td> indexwriter_memory
      </td>
      <td>索引作者在字节中估计的内存使用情况。
      </td>
    </tr>
    <tr>
      <td> bitset_memory
      </td>
      <td>字节中的缓存位集的估计内存使用量。
      </td>
    </tr>
    <tr>
      <td>版本map_memory
      </td>
      <td>字节中版本映射的估计内存使用情况。
      </td>
    </tr>
    <tr>
      <td> shard_size_in_bytes
      </td>
      <td>估计字节中碎片的磁盘使用情况。
      </td>
    </tr>
  </tbody>
</table>

## 相关维度：`ShardID`，`IndexName`，`IndexingStage`  
  
<table>
  <thead样式="text-align: left">
   <tr>
    <th>公制</th>
    <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td> indexing_pressure_current_limits
      </td>
      <td>在特定索引阶段（协调，主或副本）中，索引碎片可用于索引碎片的总堆大小。
      </td>
    </tr>
    <tr>
      <td> indexing_pressure_current_bytes
      </td>
      <td>在特定索引阶段（协调，主或副本）在特定索引阶段占据的总堆大小，在字节中。
      </td>
    </tr>
    <tr>
      <td> indexing_pressure_last_successful_timestamp
      </td>
      <td>在特定索引阶段（协调，主或副本）中成功请求索引碎片的时间戳。
      </td>
    </tr>
    <tr>
      <td>索引
      </td>
      <td>在特定索引阶段（协调，主或副本），OpenSearch对索引碎片执行的拒绝总数。
      </td>
    </tr>
    <tr>
      <td>索引
      </td>
      <td>最后n请求的平均吞吐量（n的值由`shard_indexing_pressure.secondary_parameter.throughput.request_size_window` 设置）在特定索引阶段（协调，主或副本）处的索引碎片。
      </td>
    </tr>
   </tbody>
 </table>
    
## 相关维度：`Operation`，`Exception`，`Indices`，`HTTPRespCode`，`ShardID`，`IndexName`，`ShardRole`    
   
 <table>
  <thead样式="text-align: left">
    <tr>
     <th>公制</th>
     <th>描述</th>
    </tr>
   </thead>
   <tbody>
    <tr>
     <td>延迟
      </td>
      <td>请求的延迟（毫秒）。
      </td>
    </tr>
   </tbody>
 </table>

## 相关维度：`MemType`   
   
 <table>
   <thead样式="text-align: left">
    <tr>
      <th>公制</th>
      <th>描述</th>
    </tr>
   </thead>
   <tbody>
    <tr>
      <td> gc_collection_event
      </td>
      <td>过去五秒钟内发生的垃圾收集数量。
      </td>
    </tr>
    <tr>
      <td> gc_collection_time
      </td>
      <td>在过去五秒钟内发生的所有垃圾收集的大约累积时间（毫秒）。
      </td>
    </tr>
    <tr>
      <td> HEAP_COMMENT
      </td>
      <td>供JVM使用的内存（字节）。
      </td>
    </tr>
    <tr>
      <td> HEAP_INIT
      </td>
      <td> JVM最初从操作系统请求内存管理的内存（字节）。
      </td>
    </tr>
    <tr>
      <td> HAEP_MAX
      </td>
      <td>可用于内存管理的最大内存（字节）。
      </td>
    </tr>
    <tr>
      <td> hape_used
      </td>
      <td>字节中使用的内存的量。
      </td>
    </tr>
   </tbody>
  </table>

## 相关维度：`DiskName`   
   
 <table>
   <thead样式="text-align: left">
    <tr>
      <th>公制</th>
      <th>描述</th>
    </tr>
   </thead>
   <tbody>
    <tr>
      <td> disk_utilization
      </td>
      <td>磁盘利用率：过去五秒钟在OpenSearch过程中所花费的磁盘时间的百分比。
      </td>
    </tr>
    <tr>
      <td> disk_waittime
      </td>
      <td>在过去五秒钟内，读写操作的平均持续时间（毫秒）。
      </td>
    </tr>
    <tr>
      <td> disk_servicerate
      </td>
      <td>服务率：MB在过去五秒钟内每秒读取或书写。该度量假定每个磁盘扇区存储512个字节。
      </td>
    </tr>
  </tbody>
</table>

## 相关维度：`DestAddr`   
   
 <table>
  <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
     <td> net_tcp_numflows
     </td>
     <td>收集的样品数量。性能分析仪每5秒收集1个样本。
     </td>
   </tr>
   <tr>
     <td> net_tcp_txq
     </td>
     <td>发送缓冲区中TCP数据包的平均数量。
     </td>
   </tr>
   <tr>
     <td> net_tcp_rxq
     </td>
     <td>接收缓冲区中TCP数据包的平均数量。
     </td>
   </tr>
   <tr>
     <td> net_tcp_lost
     </td>
     <td>未恢复的重复超时的平均数量。恢复完成或`SND.UNA` 是先进的。`SND.UNA` 是已发送但尚未确认的数据字节的序列编号。
     </td>
  </tr>
  <tr>
     <td> net_tcp_sendcwnd
     </td>
     <td>发送拥塞窗口的平均大小，字节。
     </td>
   </tr>
   <tr>
     <td> net_tcp_ssthresh
     </td>
     <td>较慢的开始尺寸阈值的平均大小，字节。
     </td>
   </tr>
 </tbody>
</table>

## 相关维度：`Direction`    
   
 <table>
   <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
   </thead>
   <tbody>
    <tr>
     <td> net_packetrate4
     </td>
     <td> IPv4数据报的总数每秒从/接收到接口，包括错误的传输或接收到的。
     </td>
    </tr>
    <tr>
      <td> net_packetdraprate4
      </td>
      <td>每秒发出或收到的IPv4数据报的总数。
      </td>
    </tr>
    <tr>
      <td> net_packetrate6
      </td>
      <td>每秒从接口传输或接收到的IPv6数据报的总数，包括错误或接收到的误差。
      </td>
    </tr>
    <tr>
      <td> net_packetwratrate6
      </td>
      <td>每秒发出或接收到的IPv6数据报的总数。
      </td>
    </tr>
    <tr>
      <td> net_throughtup
      </td>
      <td>所有网络接口每秒传输或接收的位数。
      </td>
    </tr>
   </tbody>
  </table>


## 相关维度：`ThreadPoolType`   
   
 <table>
   <thead样式="text-align: left">
    <tr>
     <th>公制</th>
     <th>描述</th>
    </tr>
  </thead>
  <tbody>
   <tr>
     <td> threadpool_queuesize
     </td>
     <td>任务队列的大小。
     </td>
   </tr>
   <tr>
     <td> threadpool_rejectedreqs
     </td>
     <td>被拒绝的执行的数量。
     </td>
   </tr>
   <tr>
     <td> threadpool_totalthreads
     </td>
     <td>池中的当前线程数。
     </td>
   </tr>
   <tr>
     <td> threadpool_activethreads
     </td>
     <td>主动执行任务的线程数量的近似数。
     </td>
   </tr>
   <tr>
     <td> threadpool_queuelatency
     </td>
     <td>任务队列的延迟。
     </td>
   </tr>
   <tr>
     <td> threadpool_queuecapacity
     </td>
     <td>任务队列的当前容量。
     </td>
    </tr>
  </tbody>
</table>

## 相关维度：`ClusterManager_PendingTaskType`  
   
 <table>
  <thead样式="text-align: left">
   <tr>
    <th>公制</th>
    <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
     <td> clustermanager_pendingqueuesize
     </td>
     <td>集群状态更新线程中的当前待处理任务数。每个节点都有一个集群状态更新线程，该线程提交群集状态更新任务，例如创建索引，更新映射，分配碎片和失败shard。
     </td>
   </tr>
  </tbody>
 </table>

## 相关维度：`Operation`，`Exception`，`Indices`，`HTTPRespCode`   
   
<table>
  <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
    <td> http_requestdocs
    </td>
    <td>请求中的项目数（仅针对`_bulk` 请求类型）。
    </td>
  </tr>
  <tr>
    <td> http_totalrequests
    </td>
    <td>在过去5秒内完成的请求数。
    </td>
  </tr>
  </tbody>
</table>

## 相关维度：`CBType` 
  
<table>
  <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
     <td> cb_estimatedsize
     </td>
     <td>当前的估计字节数。
     </td>
    </tr>
    <tr>
     <td> cb_trippedevents
     </td>
     <td>断路器绊倒的次数。
     </td>
    </tr>
    <tr>
      <td> cb_configuredsize
      </td>
      <td>在字节中的限制可以使用内存操作的量。
      </td>
    </tr>
   </tbody>
  </table>

## 相关维度：`ClusterManagerTaskInsertOrder`，`ClusterManagerTaskPriority`，`ClusterManagerTaskType`，`ClusterManagerTaskMetadata`
 
<table>
 <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td> clustermanager_task_queue_time
     </td>
     <td>千分之一的时间是集群管理器在队列中花费的时间。
     </td>
   </tr>
   <tr>
      <td> clusterManager_task_run_time
      </td>
      <td>千分之一的时间是集群管理器任务正在运行的时间。
      </td>
    </tr>
 </tbody>
</table>
     
## 相关维度：`CacheType` 
  
<table>
  <thead样式="text-align: left">
    <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
     <td> cache_maxsize
     </td>
     <td>高速缓存的最大大小，字节。
     </td>
   </tr>
 </tbody>
</table>

## 相关维度：`ControllerName` 
<table>
 <thead样式="text-align: left">
  <tr>
    <th>公制</th>
    <th>描述</th>
  </tr>
 </thead>
 <tbody>
  <tr>
    <td> gensission control_dectiondCount
    </td>
    <td>由入院控制器控制器执行的拒绝总数。
    </td>
  </tr>
  <tr>
    <td> gensission control_currentvalue
    </td>
    <td>入院控制器控制器的当前值。
    </td>
  </tr>
  <tr>
    <td>录取control_thresholdvalue
    </td>
    <td>入学控制控制器控制器的阈值。
    </td>
  </tr>
 </tbody>
</table>

## 相关维度：`NodeID` 
  
<table>
  <thead样式="text-align: left">
   <tr>
   <th>公制</th>
   <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td> data_retryingpendingTaskScount
      </td>
      <td>数据节点正在积极执行重试的节流待处理任务的数量。在那个时间点，这是绝对的度量。
      </td>
    </tr>
    <tr>
      <td> clustermanager_throttledpendertaskscount
      </td>
      <td>群集管理器节点限制的总待处理任务的总和。这是一个累积度量，因此请确保检查最大聚合。
      </td>
    </tr>
  </tbody>
 </table>

## 相关维度：不/a
以下指标与整个集群有关，不需要特定的维度。

<table>
  <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
     <td> eprection_term
     </td>
     <td>每个集群经理选举中单调增加的数字。
     </td>
   </tr>
   <tr>
     <td> PublishClusterState_latency
     </td>
     <td>节点法定人数发布新群集状态的时间。该指标可用于当前集群管理器。
     </td>
   </tr>
   <tr>
     <td> PublishClusterState_failure
     </td>
     <td>新群集状态未能在群集管理器节点上发布的次数。
     </td>
   </tr>
   <tr>
     <td> clusterApplierService_latency
     </td>
     <td>每个节点为群集管理器发送的应用群集状态所花费的时间。
     </td>
   </tr>
   <tr>
     <td> clusterApplierService_failure
     </td>
     <td>每个节点上应用群集状态操作失败的次数。
     </td>
   </tr>
  </tbody>
 </table>

## 相关维度：`IndexName`，`NodeName`，`ShardType`，`ShardID`
  
<table>
   <thead样式="text-align: left">
   <tr>
     <th>公制</th>
     <th>描述</th>
   </tr>
  </thead>
  <tbody>
   <tr>
     <td> shard_state
     </td>
     <td>每个碎片的状态，例如`STARTED`，`UNASSIGNED`， 或者`RELOCATING`。
     </td>
   </tr>
   </tbody>
</table>


## 维度参考

| 方面| 返回值|
|----------------------|-------------------------------------------------|
| Shardid| 例如，碎片的ID`1`。|
| indexname| 索引的名称，例如`my-index`。|
| 手术| 例如，操作类型`shardbulk`。|
| 碎片| 例如，碎片角色`primary` 或者`replica`。|
| 例外| openSearch例外，例如`org.opensearch.index_not_found_exception`。|
| 指数| 请求URL中的索引列表。|
| httprespcode| 例如，OpenSearch的响应代码，例如`200`。|
| memtype| 例如，内存类型`totYoungGC`，`totFullGC`，`Survivor`，`PermGen`，`OldGen`，`Eden`，`NonHeap`， 或者`Heap`。|
| 不解体| 例如，磁盘的名称，例如`sda1`。|
| Destaddr| 例如，目标地址`010015AC`。|
| 方向| 例如，方向`in` 或者`out`。|
| 线程pooltype| 例如，OpenSearch线程池`index`，`search`， 或者`snapshot`。|
| cbtype| 例如，断路器类型`accounting`，`fielddata`，`in_flight_requests`，`parent`， 或者`request`。|
| ClusterManagerTaskInsertorder| 例如，插入任务的顺序`3691`。|
| clusterManagerTaskPriority| 任务的优先级，例如`URGENT`。OpenSearch执行更高-较低之前的优先任务-优先的，不管`insert_order`。|
| clusterManagerTaskType| 例如，任务类型`shard-started`，`create-index`，`delete-index`，`refresh-mapping`，`put-mapping`，`CleanupSnapshotRestoreState`， 或者`Update snapshot state`。|
| clustermanagertaskmetadata| 任务的元数据（如果有）。|
| Cachetype| 例如，缓存类型，例如`Field_Data_Cache`，`Shard_Request_Cache`， 或者`Node_Query_Cache`。|



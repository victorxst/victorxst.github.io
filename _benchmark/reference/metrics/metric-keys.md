---
layout: default
title: 指标键
nav_order: 35
parent: 指标参考
grand_parent: OpenSearch基准参考
redirect_from: /benchmark/metrics/metric-keys/
---

# 指标键

公制密钥是基于在[指标记录]({{site.url}}{{site.baseurl}}/benchmark/metrics/metric-records/)。OpenSearch Benchmark存储以下指标：


- `latency`：提交请求和收到完整响应之间的时间段。这还包括等待时间，例如请求花费的时间等待，直到准备好由OpenSearch Benchmark服务为止。
- `service_time`：发送请求和接收相应响应之间的时间段。该指标类似于延迟，但不包括等待时间。
- `processing_time`：开始处理请求与接收完整响应之间的时间段。与服务时间相反，该指标还包括OpenSearch基准客户端-侧处理开销。服务时间和处理时间之间的很大差异表示客户端的高间接开销，因此可以指向潜在的客户-侧面瓶颈，需要调查。
- `throughput`：OpenSearch基准测试的操作数量可以在特定时间段内（通常每秒）执行。看到[工作负载参考]({{site.url}}{{site.baseurl}}/benchmark/workloads/index/) 用于操作类型的定义。
- `disk_io_write_bytes`：在基准期间写入磁盘的字节数。在Linux上，该度量仅对应于OpenSearch基准测试的字节。在Mac OS上，它包括所有过程编写的字节数。
- `disk_io_read_bytes`：基准期间从磁盘读取的字节数。在MacOS上，其中包括所有过程编写的字节数。
- `node_startup_time`：从过程开始到节点运行的时间，几秒钟内。
- `node_total_young_gen_gc_time`：年轻人的总运行时间-如节点统计API报道，整个集群中的一代垃圾收集器。
- `node_total_young_gen_gc_count`：年轻的总数-如节点统计API报道，整个集群的一代垃圾收集。
- `node_total_old_gen_gc_time`：旧的总运行时间-如节点统计API报道，整个集群中的一代垃圾收集器。
- `node_total_old_gen_gc_count`：旧的总数-如节点统计API报道，整个集群的一代垃圾收集。
- `node_total_zgc_cycles_gc_time`：Z垃圾收集器（ZGC）在整个集群中收集垃圾的总时间，如节点统计API所报道。
- `node_total_zgc_cycles_gc_count`：如节点统计API报道，ZGC在整个集群中执行的垃圾集合总数。
- `node_total_zgc_pauses_gc_time`：ZGC在停止中花费的总时间-这-如节点统计API报道，全世界在整个集群中停顿。
- `node_total_zgc_pauses_gc_count`：停止总数-这-如节点统计API所报道，在整个集群中ZGC执行期间的世界停顿。
- `segments_count`：索引统计API报告的开放片段总数。
- `segments_memory_in_bytes`：索引统计信息API报告，用于所有开放段的字节总数。
- `segments_doc_values_memory_in_bytes`：用于文档值的字节数量，如索引统计API所报告。
- `segments_stored_fields_memory_in_bytes`：如索引统计API报道，用于存储字段的字节数。
- `segments_terms_memory_in_bytes`：索引统计信息API报告的字节数量。
- `segments_norms_memory_in_bytes`：索引统计信息API报告，用于规范的字节数。
- `segments_points_memory_in_bytes`：用于点的字节数，如索引统计API所报告。
- `merges_total_time`：根据索引统计API的报道，主要碎片合并的累积运行时间。请注意，这次不是壁时钟时间。如果M合并线程持续了N分钟，则基准将时间报告为m * n分钟，而不是n分钟。这些指标记录还有一个额外-碎片属性包含阵列中主要碎片的时间。
- `merges_total_count`：根据指数统计API的报道，主要碎片的累积合并数量`_all/primaries`。
- `merges_total_throttled_time`：根据索引统计数据的报道，累积的合并累积时间。请注意，这次不是壁时钟时间。这些指标记录还有一个额外-碎片属性包含阵列中主要碎片的时间。
- `indexing_total_time`：索引统计API报道，用于主要碎片索引的累积时间。请注意，这不是壁时钟时间。这些指标记录还有一个额外-碎片属性包含阵列中主要碎片的时间。
- `indexing_throttle_time`：根据索引统计数据的报道，索引的累积时间已被限制。请注意，这不是壁时钟时间。这些指标记录还有一个额外-碎片属性包含阵列中主要碎片的时间。
- `refresh_total_time`：索引统计API报道，用于原代碎片的索引刷新的累积时间。请注意，这不是壁时钟时间。这些指标记录还有一个额外-碎片属性包含阵列中主要碎片的时间。
- `refresh_total_count`：根据索引统计API的报道，主要碎片的累积数量`_all/primaries`。
- `flush_total_time`：索引数据API报道，用于原始碎片索引冲洗的累积时间。请注意，这不是壁时钟时间。这些指标记录还有一个额外-碎片属性包含阵列中主要碎片的时间。
- `flush_total_count`：根据指数统计数据API的报道，主要碎片的冲洗数量`_all/primaries`。
- `final_index_size_bytes`：在所有节点都在基准的末尾关闭所有节点后，文件系统上的最终索引大小。它包括节点数据目录中的所有文件，例如索引文件和Translog。
- `store_size_in_bytes`：索引的大小，不包括Translog，如索引统计API所述，在字节中。
- `translog_size_in_bytes`：Translog的大小，如索引统计API所述，在字节中。
- `ml_processing_time`：以毫秒为单位的每个机器学习工作的对象，均值，平均值，中值和最大存储桶处理时间。这些指标仅在相应基准中创建机器学习工作时可用。


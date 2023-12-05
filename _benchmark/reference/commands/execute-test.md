---
layout: default
title: 执行-测试
nav_order: 65
parent: 命令参考
grand_parent: OpenSearch基准参考
redirect_from: /benchmark/commands/execute-test/
---

# 执行-测试

您是否正在使用随附的[OpenSearch基准测试工作负载](https://github.com/opensearch-project/opensearch-benchmark-workloads) 或a[定制工作量]({{site.url}}{{site.baseurl}}/benchmark/creating-custom-workloads/)， 使用`execute-test` 命令根据所选的工作负载收集有关OpenSearch Cluster的性能的数据。

## 用法

以下示例使用`geonames` 在测试模式下工作量：

```
opensearch-benchmark execute-test --workload=geonames --test-mode 
```

测试运行后，OpenSearch Benchmark以基准指标的摘要做出响应：

```
------------------------------------------------------
    _______             __   _____
   / ____(_)___  ____ _/ /  / ___/_________  ________
  / /_  / / __ \/ __ `/ /   \__ \/ ___/ __ \/ ___/ _ \
 / __/ / / / / / /_/ / /   ___/ / /__/ /_/ / /  /  __/
/_/   /_/_/ /_/\__,_/_/   /____/\___/\____/_/   \___/
------------------------------------------------------

|                         Metric |                 Task |     Value |   Unit |
|-------------------------------:|---------------------:|----------:|-------:|
|            Total indexing time |                      |   28.0997 |    min |
|               Total merge time |                      |   6.84378 |    min |
|             Total refresh time |                      |   3.06045 |    min |
|               Total flush time |                      |  0.106517 |    min |
|      Total merge throttle time |                      |   1.28193 |    min |
|               Median CPU usage |                      |     471.6 |      % |
|             Total Young Gen GC |                      |    16.237 |      s |
|               Total Old Gen GC |                      |     1.796 |      s |
|                     Index size |                      |   2.60124 |     GB |
|                  Total written |                      |   11.8144 |     GB |
|         Heap used for segments |                      |   14.7326 |     MB |
|       Heap used for doc values |                      |  0.115917 |     MB |
|            Heap used for terms |                      |   13.3203 |     MB |
|            Heap used for norms |                      | 0.0734253 |     MB |
|           Heap used for points |                      |    0.5793 |     MB |
|    Heap used for stored fields |                      |  0.643608 |     MB |
|                  Segment count |                      |        97 |        |
|                 Min Throughput |         index-append |   31925.2 | docs/s |
|              Median Throughput |         index-append |   39137.5 | docs/s |
|                 Max Throughput |         index-append |   39633.6 | docs/s |
|      50.0th percentile latency |         index-append |   872.513 |     ms |
|      90.0th percentile latency |         index-append |   1457.13 |     ms |
|      99.0th percentile latency |         index-append |   1874.89 |     ms |
|       100th percentile latency |         index-append |   2711.71 |     ms |
| 50.0th percentile service time |         index-append |   872.513 |     ms |
| 90.0th percentile service time |         index-append |   1457.13 |     ms |
| 99.0th percentile service time |         index-append |   1874.89 |     ms |
|  100th percentile service time |         index-append |   2711.71 |     ms |
|                           ...  |                  ... |       ... |    ... |
|                           ...  |                  ... |       ... |    ... |
|                 Min Throughput |     painless_dynamic |   2.53292 |  ops/s |
|              Median Throughput |     painless_dynamic |   2.53813 |  ops/s |
|                 Max Throughput |     painless_dynamic |   2.54401 |  ops/s |
|      50.0th percentile latency |     painless_dynamic |    172208 |     ms |
|      90.0th percentile latency |     painless_dynamic |    310401 |     ms |
|      99.0th percentile latency |     painless_dynamic |    341341 |     ms |
|      99.9th percentile latency |     painless_dynamic |    344404 |     ms |
|       100th percentile latency |     painless_dynamic |    344754 |     ms |
| 50.0th percentile service time |     painless_dynamic |    393.02 |     ms |
| 90.0th percentile service time |     painless_dynamic |   407.579 |     ms |
| 99.0th percentile service time |     painless_dynamic |   430.806 |     ms |
| 99.9th percentile service time |     painless_dynamic |   457.352 |     ms |
|  100th percentile service time |     painless_dynamic |   459.474 |     ms |

----------------------------------
[INFO] SUCCESS (took 2634 seconds)
----------------------------------
```

## 选项

使用以下选项自定义`execute-test` 命令您的用例。本节中的选项按其用例进行分类。

## 常规设置

以下选项塑造了每个测试的运行方式以及结果的显示方式：

- `--test-mode`：在测试模式下运行给定的工作负载，这在检查工作负载是否错误时很有用。
- `--user-tag`：定义用户-具体键-值对公制记录中的价值对作为元信息，例如`intention:baseline-ticket-12345`。
- `--results-format`：定义命令行结果的输出格式`markdown` 或者`csv`。默认为`markdown`。
- `--results-number-align`：定义列的列号对齐`compare` 命令输出结果。默认为`right`。
- `--results-file`：提供的文件路径后，将结果与路径中指示的文件进行比较。
- `--show-in-results`：确定是否在结果文件中包括比较。


### 分布

以下选项集，哪个版本的OpenSearch和OpenSearch插件基准测试使用：

- `--distribution-version`：根据版本号下载指定的OpenSearch发行版。有关已发布的OpenSearch版本列表，请参见[版本历史记录](https://opensearch.org/docs/version-history/)。
- `--distribution-repository`：定义了应下载OpenSearch发行版的存储库。默认为`release`。
- `--revision`：定义用于运行基准测试的当前源代码修订版。默认为`current`。
   - `current`：根据您的OpenSearch发行版，使用源树的当前修订版。
   - `latest`：从源树的主要分支中获取最新的修订。
   - 您也可以使用时间戳或从源树提交ID。使用时间戳时，指定`@ts`， 在哪里"ts" 是有效的ISO 8601时间戳，例如`@2013-07-27T10:37:00Z`。
-  `--opensearch-plugins`：定义哪个[OpenSearch插件]({{site.url}}{{site.baseurl}}/install-and-configure/plugins/) 安装。默认情况下，没有安装插件。
- `--plugin-params:` 定义一个逗号-键的分离列表：逐字注入所有插件的值对作为变量。
- `--runtime-jdk`：使用JDK的主要版本。
- `--client-options`：定义逗号-分开使用的客户列表。所有选项都传递给OpenSearch Python客户端。默认为`timeout:60`。

### 簇

以下选项与基准的目标群集有关。

- `--target-hosts`：定义逗号-分开的主机列表-如果使用管道，则应针对的端口对`benchmark-only`。默认为`localhost:9200`。


### 分布式工作量生成

以下选项可以帮助那些想使用多个主机为基准集群生成负载的人：

- `--load-worker-coordinator-hosts`：定义逗号-分开的主机列表，这些主机坐标负载。默认为`localhost`。
- `--enable-worker-coordinator-profiling`：启用OpenSearch基准的工人协调员的性能分析。默认为`false`。

### 供应

以下选项有助于自定义OpenSearch基准规定OpenSearch and Workloads：

- `--provision-config-repository`：定义opensearch基准加载的存储库`provision-configs` 和`provision-config-instances`。
- `--provision-config-path`：定义通往`--provision-config-instance` 以及要使用的任何OpenSearch插件配置。
- `--provision-config-revision`：在`provision-config` 该OpenSearch基准应使用。
- `--provision-config-instance`：定义`--provision-config-instance` 使用。您可以使用命令看到可能的配置实例`opensearch-benchmark list provision-config-instances`。
- `--provision-config-instance-params`：逗号-钥匙的分开列表-价值对逐字注射为变量`provision-config-instance`。


### 工作量

以下选项确定使用哪个工作负载来运行测试：

- `--workload-repository`：定义opensearch基准加载工作负载的存储库。
- `--workload-path`：定义下载或自定义工作负载的路径。
- `--workload-revision`：定义了OpenSearch基准应使用的工作负载源树的特定修订。
- `--workload`：定义要根据工作负载的名称使用的工作负载。您可以使用`opensearch-benchmark list workloads`。

### 测试程序

以下选项定义了该过程中测试使用的测试过程以及该过程中包含哪些操作：

- `--test-execution-id`：定义此测试运行的唯一ID。
- `--test-procedure`：定义使用的测试过程。您可以使用`opensearch-benchmark list test-procedures`。
- `--include-tasks`：定义逗号-分开运行的测试过程列表。默认情况下，运行测试过程中列出的所有任务均已运行。
- `--exclude-tasks`：定义逗号-分开的测试过程任务列表不运行。
- `--enable-assertions`：启用任务的断言检查。默认为`false`。

### 管道

这`--pipeline` 选项选择要运行的管道。您可以通过运行来找到由OpenSearch基准支持的管道列表`opensearch-benchmark list pipelines`。


### 遥测

以下选项可以在OpenSearch基准下启用遥测设备：
 
- `--telemetry`：当使用逗号提供设备时，启用提供的遥测设备-分开列表。您可以使用`opensearch-benchmark list telemetry`。
- `--telemetry-params`：定义逗号-钥匙的分开列表-逐字注入遥测设备的值对作为参数。


### 错误

以下选项设置了运行测试时OpenSearch Benchmark如何处理错误：

- `--on-error`：控制OpenSearch基准测试方式如何响应错误。默认为`continue`。
  - `continue`：尽管出错，仍继续进行测试。
  - `abort`：在发生错误时中止测试。
- `--preserve-install`：保持基准候选人及其指数。默认为`false`。
- `--kill-running-processes`：设置为`true`，停止当前正在运行的任何OpenSearch基准进程，并允许OpenSearch基准测试继续运行。默认为`false`。


---
layout: default
title: 快速开始
nav_order: 2
---

# OpenSearch基准QuickStart

该教程概述了如何快速安装OpenSearch基准测试并运行您的第一个OpenSearch基准工作负载。

## 先决条件

要执行快速步骤，您需要实现以下先决条件：

- 当前活动的OpenSearch集群。有关如何创建OpenSearch集群的说明，请参阅[创建一个集群]({{site.url}}{{site.baseurl}}//tuning-your-cluster/index/)。
- git 2.3或更大。
- Python 3.8或更高版本

## 设置OpenSearch集群

如果您还没有活动的OpenSearch集群，则可以启动一个新的OpenSearch群集，以与OpenSerch Benchmark一起使用。

- 使用**Docker组成**。有关如何使用Docker组成的说明，请参见[OpenSearch QuickStart]({{site.url}}{{site.baseurl}}/quickstart/)。
- 使用**柏油**。有关如何使用TAR安装OpenSearch的说明，请参阅[安装OpenSearch> TARBALL]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/tar#step-1-download-and-unpack-opensearch)。

OpenSearch基准测试尚未通过窗口的OpenSearch的分布进行测试。
{: .note}

安装后，您可以验证OpenSearch正在运行`localhost:9200`。如果您正在使用启用安全插件运行群集，则OpenSearch将期望使用用户名SSL连接"admin" 和密码"admin"。但是，由于Localhost地址不是唯一的公共地址，因此任何证书机构都不会向其签发SSL证书，因此需要使用该证书检查`-k` 选项。

使用以下命令验证OpenSearch正在使用SSL证书检查禁用：

```bash
curl -k -u admin:admin https://localhost:9200			# the "-k" option skips SSL certificate checks

{
  "name" : "147ddae31bf8.opensearch.org",
  "cluster_name" : "opensearch",
  "cluster_uuid" : "n10q2RirTIuhEJCiKMkpzw",
  "version" : {
    "distribution" : "opensearch",
    "number" : "2.10.0",
    "build_type" : "tar",
    "build_hash" : "eee49cb340edc6c4d489bcd9324dda571fc8dc03",
    "build_date" : "2023-09-20T23:54:29.889267151Z",
    "build_snapshot" : false,
    "lucene_version" : "9.7.0",
    "minimum_wire_compatibility_version" : "7.10.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

在运行群集后，您现在可以安装OpenSearch基准测试。

## 安装OpenSearch基准测试

要使用Docker安装OpenSearch基准测试，请参见[安装OpenSearch基准测试>与Docker安装]({{site.url}}{{site.baseurl}}/benchmark/user-guide/installing-benchmark/#installing-with-docker)。
{: .tip}

要从PYPI安装OpenSearch基准测试，请输入以下内容`pip` 命令：

```bash
pip3 install opensearch-benchmark
```
{％include copy.html％}

安装完成后，通过输入以下命令来验证OpenSearch基准测试：

```bash
opensearch-benchmark --help
```

如果成功，OpenSearch返回以下回复：

```bash
$ opensearch-benchmark --help
usage: opensearch-benchmark [-h] [--version] {execute-test,list,info,create-workload,generate,compare,download,install,start,stop} ...

   ____                  _____                      __       ____                  __                         __
  / __ \____  ___  ____ / ___/___  ____ ___________/ /_     / __ )___  ____  _____/ /_  ____ ___  ____ ______/ /__
 / / / / __ \/ _ \/ __ \\__ \/ _ \/ __ `/ ___/ ___/ __ \   / __  / _ \/ __ \/ ___/ __ \/ __ `__ \/ __ `/ ___/ //_/
/ /_/ / /_/ /  __/ / / /__/ /  __/ /_/ / /  / /__/ / / /  / /_/ /  __/ / / / /__/ / / / / / / / / /_/ / /  / ,<
\____/ .___/\___/_/ /_/____/\___/\__,_/_/   \___/_/ /_/  /_____/\___/_/ /_/\___/_/ /_/_/ /_/ /_/\__,_/_/  /_/|_|
    /_/

 A benchmarking tool for OpenSearch

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

subcommands:
  {execute-test,list,info,create-workload,generate,compare,download,install,start,stop}
    execute-test        Run a benchmark
    list                List configuration options
    info                Show info about a workload
    create-workload     Create a Benchmark workload from existing data
    generate            Generate artifacts
    compare             Compare two test_executions
    download            Downloads an artifact
    install             Installs an OpenSearch node locally
    start               Starts an OpenSearch node locally
    stop                Stops an OpenSearch node locally

Find out more about Benchmark at https://opensearch.org/docs
```

## 运行您的第一个基准测试

您现在可以运行第一个基准。以下基准使用[渗滤剂](https://github.com/opensearch-project/opensearch-benchmark-workloads/tree/main/percolator) 工作量。


### 了解工作负载命令标志

使用基准运行[`execute-test`]({{site.url}}{{site.baseurl}}/benchmark/commands/execute-test/) 带有以下命令标志的命令：

附加`execute_test` 命令标志，请参阅[执行-测试]({{site.url}}{{site.baseurl}}/benchmark/commands/execute-test/) 参考。一些常用的选项是`--workload-params`，`--exclude-tasks`， 和`--include-tasks`。
{: .tip}

*`--pipeline=benchmark-only` ：通知OSB用户想要提供自己的OpenSearch集群。
- `workload=percolator`：OpenSearch基准测试使用的工作量名称。
*`--target-host="<OpenSearch Cluster Endpoint>"`：指示将基准测试的目标群集或主机。在此处输入OpenSearch集群的端点。
*`--client-options="basic_auth_user:'<Basic Auth Username>',basic_auth_password:'<Basic Auth Password>'"`：您的OpenSearch cluster的用户名和密码。
*`--test-mode`：允许用户在整个持续时间内运行工作量而无需运行。当存在此标志时，基准将运行工作负载中每个任务的前一千个操作。这仅是为了理智检查---产生的指标毫无意义。

这`--distribution-version`，这表明在配置时将使用哪种OpenSearch版本基准。运行时，`execute-test` 命令将在连接到OpenSearch集群时解析正确的分发版本。

### 运行工作负载

运行[渗滤剂](https://github.com/opensearch-project/opensearch-benchmark-workloads/tree/main/percolator) 使用OpenSearch基准测试，使用以下内容`execute-test` 命令：

```bash
opensearch-benchmark execute-test --pipeline=benchmark-only --workload=percolator --target-host=https://localhost:9200 --client-options=basic_auth_user:admin,basic_auth_password:admin,verify_certs:false --test-mode
```
{％include copy.html％}

当。。。的时候`execute_test` 命令运行，所有任务和操作`percolator` 工作负载依次运行。

### 了解结果

基准完成后，基准将返回以下响应：

```bash
------------------------------------------------------
    _______             __   _____
   / ____(_)___  ____ _/ /  / ___/_________  ________
  / /_  / / __ \/ __ `/ /   \__ \/ ___/ __ \/ ___/ _ \
 / __/ / / / / / /_/ / /   ___/ / /__/ /_/ / /  /  __/
/_/   /_/_/ /_/\__,_/_/   /____/\___/\____/_/   \___/
------------------------------------------------------

|                                                         Metric |                                       Task |       Value |   Unit |
|---------------------------------------------------------------:|-------------------------------------------:|------------:|-------:|
|                     Cumulative indexing time of primary shards |                                            |     0.02655 |    min |
|             Min cumulative indexing time across primary shards |                                            |           0 |    min |
|          Median cumulative indexing time across primary shards |                                            |  0.00176667 |    min |
|             Max cumulative indexing time across primary shards |                                            |   0.0140333 |    min |
|            Cumulative indexing throttle time of primary shards |                                            |           0 |    min |
|    Min cumulative indexing throttle time across primary shards |                                            |           0 |    min |
| Median cumulative indexing throttle time across primary shards |                                            |           0 |    min |
|    Max cumulative indexing throttle time across primary shards |                                            |           0 |    min |
|                        Cumulative merge time of primary shards |                                            |   0.0102333 |    min |
|                       Cumulative merge count of primary shards |                                            |           3 |        |
|                Min cumulative merge time across primary shards |                                            |           0 |    min |
|             Median cumulative merge time across primary shards |                                            |           0 |    min |
|                Max cumulative merge time across primary shards |                                            |   0.0102333 |    min |
|               Cumulative merge throttle time of primary shards |                                            |           0 |    min |
|       Min cumulative merge throttle time across primary shards |                                            |           0 |    min |
|    Median cumulative merge throttle time across primary shards |                                            |           0 |    min |
|       Max cumulative merge throttle time across primary shards |                                            |           0 |    min |
|                      Cumulative refresh time of primary shards |                                            |   0.0709333 |    min |
|                     Cumulative refresh count of primary shards |                                            |         118 |        |
|              Min cumulative refresh time across primary shards |                                            |           0 |    min |
|           Median cumulative refresh time across primary shards |                                            |  0.00186667 |    min |
|              Max cumulative refresh time across primary shards |                                            |   0.0511667 |    min |
|                        Cumulative flush time of primary shards |                                            |  0.00963333 |    min |
|                       Cumulative flush count of primary shards |                                            |           4 |        |
|                Min cumulative flush time across primary shards |                                            |           0 |    min |
|             Median cumulative flush time across primary shards |                                            |           0 |    min |
|                Max cumulative flush time across primary shards |                                            |  0.00398333 |    min |
|                                        Total Young Gen GC time |                                            |           0 |      s |
|                                       Total Young Gen GC count |                                            |           0 |        |
|                                          Total Old Gen GC time |                                            |           0 |      s |
|                                         Total Old Gen GC count |                                            |           0 |        |
|                                                     Store size |                                            | 0.000485923 |     GB |
|                                                  Translog size |                                            | 2.01873e-05 |     GB |
|                                         Heap used for segments |                                            |           0 |     MB |
|                                       Heap used for doc values |                                            |           0 |     MB |
|                                            Heap used for terms |                                            |           0 |     MB |
|                                            Heap used for norms |                                            |           0 |     MB |
|                                           Heap used for points |                                            |           0 |     MB |
|                                    Heap used for stored fields |                                            |           0 |     MB |
|                                                  Segment count |                                            |          32 |        |
|                                                 Min Throughput |                                      index |     3008.97 | docs/s |
|                                                Mean Throughput |                                      index |     3008.97 | docs/s |
|                                              Median Throughput |                                      index |     3008.97 | docs/s |
|                                                 Max Throughput |                                      index |     3008.97 | docs/s |
|                                        50th percentile latency |                                      index |     351.059 |     ms |
|                                       100th percentile latency |                                      index |     365.058 |     ms |
|                                   50th percentile service time |                                      index |     351.059 |     ms |
|                                  100th percentile service time |                                      index |     365.058 |     ms |
|                                                     error rate |                                      index |           0 |      % |
|                                                 Min Throughput |                   wait-until-merges-finish |       28.41 |  ops/s |
|                                                Mean Throughput |                   wait-until-merges-finish |       28.41 |  ops/s |
|                                              Median Throughput |                   wait-until-merges-finish |       28.41 |  ops/s |
|                                                 Max Throughput |                   wait-until-merges-finish |       28.41 |  ops/s |
|                                       100th percentile latency |                   wait-until-merges-finish |     34.7088 |     ms |
|                                  100th percentile service time |                   wait-until-merges-finish |     34.7088 |     ms |
|                                                     error rate |                   wait-until-merges-finish |           0 |      % |
|                                                 Min Throughput |     percolator_with_content_president_bush |       36.09 |  ops/s |
|                                                Mean Throughput |     percolator_with_content_president_bush |       36.09 |  ops/s |
|                                              Median Throughput |     percolator_with_content_president_bush |       36.09 |  ops/s |
|                                                 Max Throughput |     percolator_with_content_president_bush |       36.09 |  ops/s |
|                                       100th percentile latency |     percolator_with_content_president_bush |     35.9822 |     ms |
|                                  100th percentile service time |     percolator_with_content_president_bush |     7.93048 |     ms |
|                                                     error rate |     percolator_with_content_president_bush |           0 |      % |

[...]

|                                                 Min Throughput |          percolator_with_content_ignore_me |        16.1 |  ops/s |
|                                                Mean Throughput |          percolator_with_content_ignore_me |        16.1 |  ops/s |
|                                              Median Throughput |          percolator_with_content_ignore_me |        16.1 |  ops/s |
|                                                 Max Throughput |          percolator_with_content_ignore_me |        16.1 |  ops/s |
|                                       100th percentile latency |          percolator_with_content_ignore_me |     131.798 |     ms |
|                                  100th percentile service time |          percolator_with_content_ignore_me |     69.5237 |     ms |
|                                                     error rate |          percolator_with_content_ignore_me |           0 |      % |
|                                                 Min Throughput | percolator_no_score_with_content_ignore_me |       29.37 |  ops/s |
|                                                Mean Throughput | percolator_no_score_with_content_ignore_me |       29.37 |  ops/s |
|                                              Median Throughput | percolator_no_score_with_content_ignore_me |       29.37 |  ops/s |
|                                                 Max Throughput | percolator_no_score_with_content_ignore_me |       29.37 |  ops/s |
|                                       100th percentile latency | percolator_no_score_with_content_ignore_me |     45.5703 |     ms |
|                                  100th percentile service time | percolator_no_score_with_content_ignore_me |      11.316 |     ms |
|                                                     error rate | percolator_no_score_with_content_ignore_me |           0 |      % |



--------------------------------
[INFO] SUCCESS (took 18 seconds)
--------------------------------
```

每个任务由`percolator` 工作负载代表特定的OpenSearch API操作---例如散装或搜索---该测试进行时进行了。输出摘要中的每个任务都包含以下信息：

***吞吐量：** 每秒成功的OpenSearch操作数量。
***潜伏：** 为请求所花费的时间（包括等待时间）以及基准发送和接收的响应。
***服务时间：** 不包括等待时间，为请求和基准发送和接收的响应的时间。
***错误率：** 在任务期间运行的百分比未成功或返回了200个错误代码。


## 在自己的群集上运行OpenSearch基准测试

现在您已经熟悉在集群上运行OpenSearch Benchmark，您可以使用相同`execute-test` 命令，更换以下设置。
 
  * 代替`https://localhost:9200` 使用目标群集端点。这可能是一个URI`https://search.mydomain.com` 或a`HOST:PORT` 规格。
  *如果将群集配置为基本身份验证，请用适当的凭据替换命令行中的用户名和密码。
  * 去除`verify_certs:false` 指令如果您没有指定`localhost` 作为目标群集。仅对于未设置SSL证书的集群才需要此指令。
  *如果您正在使用`HOST:PORT`规格和计划使用SSL/TLS，要么指定`https://`，或添加`use_ssl:true` 指令`--client-options` 字符串选项。
  * 去除`--test-mode` 标志以运行完整的工作量，而不是缩写测试。

您可以复制以下命令模板以在您自己的终端中使用：

```bash
opensearch-benchmark execute-test --pipeline=benchmark-only --workload=percolator --target-host=<OpenSearch Cluster Endpoint> --client-options=basic_auth_user:admin,basic_auth_password:admin
```
{％include copy.html％}

## 下一步

请参阅以下资源以了解有关OpenSearch基准测试的更多信息：

- [用户指南]({{site.url}}{{site.baseurl}}/benchmark/user-guide/index/)：深入了解如何开放搜索基准测试您可以帮助您跟踪群集的性能。
- [教程]({{site.url}}{{site.baseurl}}/benchmark/tutorials/index/)：使用步骤-经过-步骤指南，用于更高级的基准配置和功能。


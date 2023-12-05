---
layout: default
title: 配置OpenSearch基准测试
nav_order: 7
parent: 用户指南
redirect_from: /benchmark/configuring-benchmark/
---

# 配置OpenSearch基准测试

OpenSearch基准配置数据存储在`~/.benchmark/benchmark.ini`，这将自动创建第一次OpenSearch基准运行。

该文件分为以下各节，您可以根据群集的需求进行自定义。

## 元

本节包含有关配置文件的元信息。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `config.version` | 整数|  配置文件格式的版本。该属性由OpenSearch基准测试管理，不应更改。|

## 系统

本节包含当前基准环境的全球信息。此信息应在安装OpenSearch基准测试的所有机器上相同。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `env.name` | 细绳| 配置OpenSearch指标存储时，在指标文档中用作元数据的基准环境的名称。仅允许字母数字字符。默认为`local`。|
| `available.cores` | 整数| 确定可用CPU内核的数量。OpenSearch基准测试旨在创建每个核心的Asyncio事件循环，并在事件循环中均匀分配给客户。默认为群集的逻辑CPU内核数。|
| `async.debug` | 布尔| 在OpenSearch基准的Asyncio事件循环中启用调试模式。默认为`false`。|
| `passenv` | 细绳| 逗号-分开的环境变量名称列表应将其传递给处理处理。|

## 节点

本节包含节点-可以根据群集的需求自定义的特定信息。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `root.dir` | 细绳| 存储所有OpenSearch基准数据的目录。OpenSearch Benchmark假定对该目录及其所有子目录的控制。|
| `src.root.dir` | 细绳| 调用OpenSearch源代码和任何OpenSearch插件的目录。仅与来自[来源](#source)。|

## 来源

本节包含有关OpenSearch源树的更多详细信息。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `remote.repo.url` | URL| 可以查看OpenSearch的URL。默认为`https://github.com/opensearch-project/OpenSearch.git`。
| `opensearch.src.subdir` | 细绳| 相对于`src.root.dir` OpenSearch搜索树的。默认为`OpenSearch`。
| `cache` | 布尔| 启用OpenSearch的内部源工件缓存，`opensearch*.tar.gz`，以及任何插件zip文件。根据其GIT修订，将文物缓存。默认为`true`。|
| `cache.days` | 整数| 应将工件保存在源文物缓存中的天数。默认为`7`。|

## 基准

本节包含可以在OpenSearch基准数据目录中自定义的设置。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `local.dataset.cache` | 细绳| 存储基准数据集的目录。根据运行的基准，该目录可能包含数百个GB数据。默认路径是`$HOME/.benchmark/benchmarks/data`。|

## Results_publishing

本节定义了如何存储基准指标。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `datastore.type` | 细绳| 如果设置为`in-memory` 运行基准时，所有指标都保存在内存中。如果设置为`opensearch` 相反，所有指标均写入持续的指标商店，并提供数据可用于进一步分析。默认为`in-memory`。|
| `sample.queue.size` | 功能| 可以存储在OpenSearch基准中的指标样本的数量-内存队列。默认为`2^20`。| 
| 指标。request.DownSample.Factor| 整数| （默认值：1）：确定在指标存储中保存了多少服务时间和延迟样本。默认情况下，所有值都保存。例如，如果您愿意。仅保留每100个样本，指定`100`。这对于避免与许多客户的基准中的指标商店淹没度量商店很有用。默认为`1`。|
| `output.processingtime` | 布尔| 如果设置为`true`，OpenSearch在命令行报告中显示了其他度量处理时间。默认为`false`。|

### `datastore.type` 参数

什么时候`datastore.type` 被设定为`opensearch`，可以自定义以下报告设置。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `datastore.host` | IP地址| 例如，指标商店的主机名`124.340.200.22`。|
| datastore.port| 港口| 例如，指标商店的端口号`9200`。|
| `datastore.secure` | 布尔| 如果设置为`false`，OpenSearch假定HTTP连接。如果设置为true，则假定HTTPS连接。|
| `datastore.ssl.verification_mode` | 细绳| 设置为默认`full`，检查了指标商店的SSL证书。要禁用证书验证，请将此值设置为`none`。|
| `datastore.ssl.certificate_authorities` | 细绳| 确定证书当局签署证书的本地文件系统路径。
| `datastore.user` | 用户名| 设置指标商店的用户名|
| `datastore.password` | 细绳| 设置指标商店的密码。或者，可以使用此密码使用`OSB_DATASTORE_PASSWORD` 环境变量，避免将凭据存储在纯文本文件中。如果两者都定义密码，则环境变量优先于配置文件。|
| `datastore.probe.cluster_version` | 细绳| 启用自动检测指标商店的版本。默认为`true`。|
| `datastore.number_of_shards` | 整数| 主要碎片的数量`opensearch-*` 索引应该有。初始索引创建后，此设置的任何更新将仅应用于新的`opensearch-*` 索引。默认为[OpenSearch静态索引值]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index-settings/#static-index-level-index-settings)。|
| `datastore.number_of_replicas` | 整数| 数据存储中的每个主碎片的复制次数包含。初始索引创建后，此设置的任何更新将仅应用于新的`opensearch-* `索引。默认为[OpenSearch静态索引值]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index-settings/#static-index-level-index-settings)。|

### 例子

您可以使用以下示例在群集中设置报告值。

此示例在本地网络中定义了一个未受保护的指标商店：

```
[results_publishing]
datastore.type = opensearch
datastore.host = 192.168.10.17
datastore.port = 9200
datastore.secure = false
datastore.user =
datastore.password =
```

此示例定义了与自我的本地网络中指标商店的安全连接-签名证书：

```
[results_publishing]
datastore.type = opensearch
datastore.host = 192.168.10.22
datastore.port = 9200
datastore.secure = true
datastore.ssl.verification_mode = none
datastore.user = user-name
datastore.password = the-password-to-your-cluster
```

## 工作负载

本节定义了如何检索工作负载。所有密钥都使用语法读取`<<workload-repository-name>>.url`，您可以使用OpenSearch基准CLI选择该，`--workload-repository=workload-repository-name"` 选项。默认情况下，OpenSearch使用`default.url` `https://github.com/opensearch-project/opensearch-benchmark-workloads`。


## 默认值

本节定义了某些OpenSearch基准CLI参数的默认值。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `preserve_benchmark_candidate` | 布尔| 确定在基准后默认情况下保留或擦除OpenSearch安装。要保留单个基准测试的安装，请使用命令行标志`--preserve-install`。默认为`false`。

## 分布

本节定义了如何分发OpenSearch版本。

| 范围| 类型| 描述|
| ：---- | ：---- | ：---- |
| `release.cache` | 布尔| 确定新发布的OpenSearch版本是否应在本地缓存。|

## 代理配置

OpenSearch自动为您下载所有必要的代理数据，包括：

- 指定时openSearch发行版`--distribution-version=<OPENSEARCH-VERSION>`。
- OpenSearch源代码，例如指定GIT修订号时，例如`--revision=1e04b2w`。
- 从[OpenSearch GitHub存储库](https://github.com/opensearch-project/OpenSearch)。

从OpenSearch基准为0.5.0，仅`http_proxy` 得到支持。
{: .warning}

您可以使用`http_proxy` 要将OpenSearch基准测试连接到特定代理，然后将代理连接到基准工作负载。添加代理：


1. 将您的代理URL添加到外壳配置文件中：

   ```
   export http_proxy=http://proxy.proxy.org:4444/
   ```

2. 来源您的外壳配置文件，并验证代理URL是否正确设置了：

   ```
   source ~/.bash_profile ; echo $http_proxy
   ```

3. 配置Git通过使用以下命令来连接您的代理。有关更多信息，请参阅[GIT文档](https://git-scm.com/docs/git-config)。

   ```
   git config --global http_proxy $http_proxy
   ```

4. 使用`git clone` 通过使用以下命令来克隆工作负载存储库。如果代理配置正确，则克隆成功。

   ```
   git clone http://github.com/opensearch-project/opensearch-benchmark-workloads.git
   ```

5. 最后，验证OpenSearch基准测试可以通过检查`/.benchmark/logs/benchmark.log` 日志。当OpenSearch基准开始时，您应该在日志顶部看到以下内容：

    ```
    Connecting via proxy URL [http://proxy.proxy.org:4444/] to the Internet (picked up from the environment variable [http_proxy]).
    ```

## 记录

可以在OpenSearch基准测试的日志中配置`~/.benchmark/logging.json` 文件。有关如何格式化日志文件的更多信息，请参见以下Python文档：

- 对于一般提示和技巧，请使用[Python伐木食谱](https://docs.python.org/3/howto/logging-cookbook.html)。
- 有关文件格式，请参阅Python[记录配置模式](https://docs.python.org/3/library/logging.config.html#logging-config-dictschema)。
- 有关如何自定义日志输出编写位置的说明，请参阅[记录处理程序文档](https://docs.python.org/3/library/logging.handlers.html)。

默认情况下，OpenSearch基准将所有输出日志记录到`~/.benchmark/logs/benchmark.log`。









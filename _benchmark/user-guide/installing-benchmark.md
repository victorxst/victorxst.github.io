---
layout: default
title: 安装OpenSearch基准测试
nav_order: 5
parent: 用户指南
redirect_from: /benchmark/installing-benchmark/
---

# 安装OpenSearch基准测试

您可以直接在运行Linux或MACOS的主机上安装OpenSearch基准测试，也可以在任何兼容主机上的Docker容器中运行OpenSearch Benchmark。此页面为您的OpenSearch基准主机提供了一般注意事项，以及用于安装OpenSearch基准测试的说明。


## 选择适当的硬件

OpenSearch基准测试可用于为测试提供OpenSearch节点。如果您打算使用OpenSearch基准测试来在环境中提供节点，请直接在群集中的每个主机上安装OpenSearch基准测试。此外，您必须在群集中配置每个主机进行opensearch。看[安装OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/) 有关重要主机设置的指导。

请记住，当您在Docker容器中运行OpenSearch Benchmark时，OpenSearch Benchmark不能用于提供OpenSearch节点。如果要使用OpenSearch基准测试来提供节点，或者要使用OpenSearch基准守护程序分发基准工作负载，则必须使用Python和pip直接在每个主机上安装OpenSearch基准。
{： 。重要的}

选择主机时，您还应该考虑要运行哪些工作负载。要查看默认基准工作负载列表，请访问[OpenSearch-基准-工作负载](https://github.com/opensearch-project/opensearch-benchmark-workloads) github上的存储库。通常，请确保OpenSearch基准主机具有足够的免费存储空间来存储压缩数据，并且一旦安装了OpenSearch基准，就可以完全解压缩数据语料库。

如果您想用默认工作量进行基准测试，请使用下表来确定与未压缩尺寸添加压缩尺寸所需的最小最小自由空间量。

| 工作负载名称| 文档计数| 压缩大小| 未压缩的尺寸|
| ：----：| ：----：| ：----：| ：----：|
| EventData| 20,000,000| 756.0 MB| 15.3 GB|
| Geonames| 11,396,503| 252.9 MB| 3.3 GB|
| 地理点| 60,844,404| 482.1 MB| 2.3 GB|
| 地理点尺| 60,844,404| 470.8 MB| 2.6 GB|
| Geoshape| 60,523,283| 13.4 GB| 45.4 GB|
| http_logs| 247,249,096| 1.2 GB| 31.1 GB|
| 嵌套| 11,203,029| 663.3 MB| 3.4 GB|
| NOAA| 33,659,481| 949.4 MB| 9.0 GB|
| nyc_taxis| 165,346,692| 4.5 GB| 74.3 GB|
| 渗滤剂| 2,000,000| 121.1 kb| 104.9 MB|
| PMC| 574,199| 5.5 GB| 21.7 GB|
| 所以| 36,062,278| 8.9 GB| 33.1 GB|

您的OpenSearch基准主机应使用固体-国家驱动器（SSD）用于存储-磁盘硬盘驱动器。旋转-磁盘硬盘驱动器可以引入性能瓶颈，这可能会使基准结果不可靠且不一致。
{: .tip}

## 在Linux和MacOS上安装

如果要在Docker容器中运行OpenSearch Benchmark，请参阅[使用Docker安装](#installing-with-docker)。OpenSearch基准Docker映像包含所有必需的软件，因此无需其他步骤。
{： 。重要的}

要直接在Unix主机（例如Linux或MacOS）上安装OpenSearch基准测试，请确保您有**Python 3.8或更高版本** 安装。

如果您需要安装Python的帮助，请参考官方[Python设置和用法](https://docs.python.org/3/using/index.html) 文档。

### 检查软件依赖性

在开始安装OpenSearch基准测试之前，请检查以下软件依赖项。

使用[Pyenv](https://github.com/pyenv/pyenv) 在主机上管理多个版本的Python。如果您的"system" Python的版本早于版本3.8。
{: .tip}

- 检查是否安装了Python 3.8或以后：

  ```bash
  python3 --version
  ```
  {% include copy.html %}

- 检查一下`pip` 已安装和功能：

  ```bash
  pip --version
  ```
  {% include copy.html %}

- _optional_：检查您的安装版本是否`git` 是**git 1.9或更高版本** 使用以下命令。`git` OpenSearch基准安装并不需要，但是在要执行测试时，需要从存储库中获取基准工作负载资源。看到官方git[文档](https://git-scm.com/doc) 为了帮助安装git。

  ```bash
  git --version
  ```
  {% include copy.html %}

### 完成安装

安装所需软件后，您可以使用以下命令安装OpenSearch基准：

```bash
pip install opensearch-benchmark
```
{% include copy.html %}

安装完成后，您可以使用以下命令显示帮助信息：

```bash
opensearch-benchmark -h
```
{% include copy.html %}


现在，OpenSearch基准已安装在主机上，您可以了解[配置OpenSearch基准测试]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark/)。

## 使用Docker安装

您可以在OpenSearch Benchmark上找到官方的Docker图像[Docker Hub](https://hub.docker.com/r/opensearchproject/opensearch-benchmark) 或在[亚马逊ECR公共画廊](https://gallery.ecr.aws/opensearchproject/opensearch-benchmark)。


### Docker限制

当您在Docker容器中运行OpenSearch Benchmark时，某些OpenSearch基准功能不可用。具体而言，以下限制适用：

- OpenSearch Benchmark无法从多个主机（例如负载工人协调器主机）分发负载。
- OpenSearch基准测试无法提供OpenSearch节点，并且只能在先前现有的群集上进行测试。您只能使用该基准命令调用OpenSearch基准命令`benchmark-only` 管道。

### 拉码头图像

要从Docker Hub中汲取图像，请运行以下命令：

```bash
docker pull opensearchproject/opensearch-benchmark:latest
```
{% include copy.html %}

从Amazon弹性容器注册表（Amazon ECR）中提取图像：

```bash
docker pull public.ecr.aws/opensearchproject/opensearch-benchmark:latest
```
{% include copy.html %}

### 与Docker一起运行基准

要运行OpenSearch基准测试，请使用`docker run` 启动一个容器。启动容器时，OpenSearch基准子标准子命令将作为参数传递。然后，OpenSearch基准测试处理命令并在请求操作完成后停止容器。

例如，以下命令在命令行中打印为OpenSearch基准测试的帮助文本，然后停止容器：

```bash
docker run opensearchproject/opensearch-benchmark -h
```
{% include copy.html %}


### 在Docker容器中建立音量持久性

为了确保在Docker容器停止之后确保您的基准数据和日志持续存在，请在使用OpenSearch基准测试时指定码头音量以安装到图像。

使用`-v` 选项，在附加卷的容器中指定安装的本地目录和一个目录。

以下示例命令在用户的主目录中创建卷`/opensearch-benchmark/.benchmark`，然后使用Geonames Workload运行测试基准。还指定了一些客户选项：

```bash
docker run -v $HOME/benchmarks:/opensearch-benchmark/.benchmark opensearchproject/opensearch-benchmark execute_test --target-hosts https://198.51.100.25:9200 --pipeline benchmark-only --workload geonames --client-options basic_auth_user:admin,basic_auth_password:admin,verify_certs:false --test-mode
```
{% include copy.html %}

看[配置OpenSearch基准测试]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark/) 要了解有关文件和子目录的更多信息`/opensearch-benchmark/.benchmark`。

## 用测试配置OpenSearch集群

OpenSearch基准测试与JDK版本17、16、15、14、13、12、11和8兼容。
{: .note}

如果您使用PYPI安装了OpenSearch，则还可以通过指定一个`distribution-version` 在里面`execute-test` 命令。

如果您打算为基准提供一个集群，则需要将基准告知基准。`JAVA_HOME` 基准群集的路径。设置`JAVA_HOME` 路径和配置集群：

1. 找出`JAVA_HOME` 您当前正在使用的路径。打开终端并输入`/usr/libexec/java_home`。

2. 通过从上一步输入路径来设置您的相应JDK版本环境变量。进入`export JAVA17_HOME=<Java Path>`。

3. 跑过`execute-test` 命令并指示您要使用的OpenSearch的发行版：

  ```bash
  opensearch-benchmark execute-test --distribution-version=2.3.0 --workload=geonames --test-mode 
  ```

## 目录结构

首次运行OpenSearch基准测试后，您可以在所有相关文件（包括配置文件）中搜索`~/.benchmark` 目录。该目录包括以下文件树：

```
# ~/.benchmark Tree
.
├── benchmark.ini
├── benchmarks
│   ├── data
│   │   └── geonames
│   ├── distributions
│   │   ├── opensearch-1.0.0-linux-x64.tar.gz
│   │   └── opensearch-2.3.0-linux-x64.tar.gz
│   ├── test_executions
│   │   ├── 0279b13b-1e54-49c7-b1a7-cde0b303a797
│   │   └── 0279c542-a856-4e88-9cc8-04306378cd38
│   └── workloads
│       └── default
│           └── geonames
├── logging.json
├── logs
│   └── benchmark.log
```

*`benchmark.ini`：包含用于测试的任何可调节配置。有关如何配置OpenSearch基准测试的信息，请参阅[配置OpenSearch基准测试]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark/)。
*`data`：包含与OpenSearch基准测试的所有数据语料库和文档[官方工作负载](https://github.com/opensearch-project/opensearch-benchmark-workloads/tree/main/geonames)。
*`distributions`：包含从[opensearch.org](http://opensearch.org/) 并用于提供簇。
*`test_executions`：包含所有测试`execution_id`从先前运行的OpenSearch基准测试中。
*`workloads`：包含与工作负载相关的所有文件，除了数据语料库。
*`logging.json`：包含与OpenSearch基准中如何执行记录有关的所有配置选项。
*`logs`：包含来自OpenSearch基准测试的所有日志运行。当您在运行过程中遇到错误时，这可能会有所帮助。


## 下一步

- [配置OpenSearch基准测试]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark/)
- [创建自定义工作负载]({{site.url}}{{site.baseurl}}/benchmark/creating-custom-workloads/)


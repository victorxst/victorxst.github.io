---
layout: default
title: 实验性功能标志
parent: 配置 OpenSearch
nav_order: 120
---

# 实验性功能标志

OpenSearch 版本可能包含实验性功能，你可以根据需要启用或禁用这些功能。有几种方法可以启用功能标志，具体取决于安装类型。

## 在 opensearch.yml 中启用

如果你正在运行 OpenSearch 集群并希望在配置文件中启用功能标志，请将以下行添加到 `opensearch.yml`：

```yaml
opensearch.experimental.feature.<feature_name>.enabled: true
```
{% include copy.html %}

## 在 Docker 容器上启用

如果运行的是 Docker，请在“> `environment`”部分下 `opensearch-node` 添加以下行 `docker-compose.yml`：

```bash
OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.<feature_name>.enabled=true"
```
{% include copy.html %}

## 在 tarball 安装上启用

要在 tarball 安装上启用功能部件标志，请在或 `OPENSEARCH_JAVA_OPTS` 中提供新的 JVM 参数 `config/jvm.options`。

### 选项 1：修改 jvm.options

在开始 `opensearch` 该过程以启用该功能及其依赖项之前，请添加以下行 `config/jvm.options`：

```bash
-Dopensearch.experimental.feature.<feature_name>.enabled=true
```
{% include copy.html %}

然后运行 OpenSearch：

```bash
./bin/opensearch
```
{% include copy.html %}

### 选项 2：使用环境变量启用

作为直接修改 `config/jvm.options` 的替代方法，你可以使用环境变量定义属性。这可以在启动 OpenSearch 时使用单个命令来完成，也可以通过使用 `export` 定义变量来完成。

要在启动 OpenSearch 时以内联方式添加功能标志，请运行以下命令：

```bash
OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.<feature_name>.enabled=true" ./opensearch-{{site.opensearch_version}}/bin/opensearch
```
{% include copy.html %}

如果要在运行 OpenSearch 之前单独定义环境变量，请运行以下命令：

```bash
export OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.<feature_name>.enabled=true"
```
{% include copy.html %}

```bash
./bin/opensearch
```
{% include copy.html %}

## 启用 OpenSearch 开发

要为开发启用功能标志，你必须在构建 OpenSearch 之前添加正确的属性 `run.gradle`。[开发者指南](https://github.com/opensearch-project/OpenSearch/blob/main/DEVELOPER_GUIDE.md)有关如何使用 Gradle 构建 OpenSearch 的信息，请参阅。

将以下属性添加到 run.gradle 以启用该功能：

```gradle
testClusters {
    runTask {
      testDistribution = 'archive'
      if (numZones > 1) numberOfZones = numZones
      if (numNodes > 1) numberOfNodes = numNodes
      systemProperty 'opensearch.experimental.feature.<feature_name>.enabled', 'true'
    }
  }
```
{% include copy.html %}
---
layout: default
title: 下载
nav_order: 60
parent: 命令参考
grand_parent: OpenSearch基准参考
redirect_from: /benchmark/commands/download/
---

# 下载

使用`download` 命令选择要下载的OpenSearch发行版本。

## 用法

以下示例下载OpenSearch版本2.7.0：

```
opensearch-benchmark download --distribution-version=2.7.0 
```
 
然后，基准返回OpenSearch文物的位置：

```
{
  "opensearch": "/Users/.benchmark/benchmarks/distributions/opensearch-2.7.0.tar.gz" 
}
```

## 选项
 
使用以下选项自定义OpenSearch基准下载OpenSearch：

- `--provision-config-repository`：定义opensearch基准加载的存储库`provision-configs` 和`provision-config-instances`。
- `--provision-config-revision`：在`provision-config` 该OpenSearch基准应使用。
- `--provision-config-path`：定义通往`--provision-config-instance` 以及要使用的任何OpenSearch插件配置。
- `--distribution-version`：根据版本号下载指定的OpenSearch发行版。有关已发布的OpenSearch版本列表，请参见[版本历史记录](https://opensearch.org/docs/version-history/)。
- `--distribution-repository`：定义了应下载OpenSearch发行版的存储库。默认为`release`。
- `--provision-config-instance`：定义`--provision-config-instance` 使用。您可以使用命令查看可能的配置实例`opensearch-benchmark list provision-config-instances`。
- `--provision-config-instance-params`：逗号-钥匙的分开列表-价值对逐字注射为变量`provision-config-instance`。
- `--target-os`：应下载OpenSearch工件的目标操作系统（OS）。默认值是当前操作系统。
- `--target-arch`：应下载工件的CPU体系结构的名称。


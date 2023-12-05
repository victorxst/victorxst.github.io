---
layout: default
title: 语料库
parent: 工作负载参考
grand_parent: OpenSearch基准参考
nav_order: 70
redirect_from: /benchmark/workloads/corpora/
---

# 语料库

这`corpora` 元素包含工作负载使用的所有文档语料库。您可以通过复制和粘贴任何CORPORA定义来跨工作负载使用文档Corpora。

## 例子

以下示例定义了一个名为`movies` 和`11658903` 文件和`1544799789` 未压缩字节：

```json
  "corpora": [
    {
      "name": "movies",
      "documents": [
        {
          "source-file": "movies-documents.json",
          "document-count": 11658903, # Fetch document count from command line
          "uncompressed-bytes": 1544799789 # Fetch uncompressed bytes from command line
        }
      ]
    }
  ]
```

## 配置选项

使用以下选项与`corpora`。

范围| 必需的| 类型| 描述
:--- | :--- | :--- | :---
`name` | 是的| 细绳| 文档语料库的名称。由于OpenSearch基准测试在其目录中使用此名称，因此仅使用没有白色空间的小写名称。
`documents` | 是的| Json Array| 一系列文档文件。
`meta` | 不| 细绳| 钥匙的映射-具有额外元数据的价值对。


每个条目`documents` 数组由以下选项组成。

范围| 必需的| 类型| 描述
:--- | :--- | :--- | :---
`source-file` | 是的| 细绳| 文件名包含工作负载的相应文档。在本地使用OpenSearch基准测试时，文档包含在JSON文件中。提供一个`base_url`，使用压缩文件格式：`.zip`，`.bz2`，`.gz`，`.tar`，`.tar.gz`，`.tgz`， 或者`.tar.bz2`。压缩文件必须具有一个包含名称的JSON文件。
`document-count` | 是的| 整数| 文档数量`source-file`，确定哪个客户索引与文档语料库的哪些部分相关。每个n客户端都会收到文档语料库的nth。当使用包含父母文档的源时-儿童关系，指定家长文档的数量。
`base-url` | 不| 细绳| HTTP（S），Amazon简单存储服务（Amazon S3）或Google Cloud Storage URL指向OpenSearch基准测试的根路径可以获得相应的源文件。
`source-format` | 不| 细绳| 定义格式OpenSearch基准测试用来解释在`source-file`。仅有的`bulk` 得到支持。
`compressed-bytes` | 不| 整数| 压缩源文件的大小（以字节为单位），指示了多少数据OpenSearch基准下载。
`uncompressed-bytes` | 不| 整数| 解压缩后源文件的大小，以字节为单位，指示解压缩源文件需要多少磁盘空间。
`target-index` | 不| 细绳| 定义索引的名称`bulk` 操作应针对目标。当在仅定义一个索引时，OpenSearch Benchmark会自动派生此值`indices` 元素。的价值`target-index` 当`includes-action-and-meta-data` 设置为`true`。
`target-type` | 不| 细绳| 定义针对批量操作的目标索引的文档类型。当在仅定义一个索引时，OpenSearch Benchmark会自动派生此值`indices` 元素和索引只有一种类型。的价值`target-type` 当`includes-action-and-meta-data` 设置为`true`。
`includes-action-and-meta-data` | 不| 布尔| 设置为`true`，指示文档的文件已经包含一个`action` 线和`meta-data` 线。什么时候`false`，指示文档的文件仅包含文档。默认为`false`。
`meta` | 不| 细绳| 钥匙的映射-具有额外元数据的价值对。



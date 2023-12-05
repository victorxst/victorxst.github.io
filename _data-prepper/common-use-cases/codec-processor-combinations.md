---
layout: default
title: 编解码器处理器组合
parent: 常见用例
nav_order: 25
---

# 编解码器处理器组合

在摄入时，收到的数据由[`s3` 来源]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3/) 可以通过[编解码器]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3#codec)。编解码器在通过数据Prepper管道摄入之前，以某种格式压缩和解压缩大型数据集[处理器]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/processors/)。

尽管大多数编解码器都可以与大多数处理器一起使用，但以下编解码器处理器组合可以使您的管道在与以下输入类型一起使用时更有效。

## Json Array

A[Json Array](https://json-schema.org/understanding-json-schema/reference/array) 用于订购不同类型的元素。因为在JSON中需要数组，所以数组中包含的数据必须是表格。

JSON数组不需要处理器。

## NDJSON

与JSON阵列不同，[NDJSON](https://www.npmjs.com/package/ndjson) 允许每行数据由Newline界定，这意味着数据是每行处理而不是数组的。

使用NDJSON输入类型使用[新队]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3#newline-codec) 编解码器，将每行解析为一个日志事件。这[parse_json]({{site.url}}{{site.baseurl}}data-prepper/pipelines/configuration/processors/parse-json/) 然后，处理器将每行作为一个事件输出。

## CSV

CSV数据类型将数据作为表输入。它可以在没有编解码器或处理器的情况下使用，但是它确实需要一个或另一个`csv` 处理器或`csv` 编解码器。

与以下编解码器处理器组合一起使用时，CSV输入类型最有效。

### `csv` 编解码器

当。。。的时候[`csv` 编解码器]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3#csv-codec) 它在没有处理器的情况下使用，它会自动检测到CSV的标题并将其用于索引映射。

### `newline` 编解码器

这[`newline` 编解码器]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3#newline-codec) 将每一行解析为一个日志事件。编解码器仅在`header_destination` 已配置。这[CSV]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/csv/) 然后，处理器将事件输出到列中。在`header_destination` 来自`newline` 编解码器可以在`csv` 处理器下`column_names_source_key.`

## 镶木

[Apache Parquet](https://parquet.apache.org/docs/overview/) 是为Hadoop构建的柱状存储格式。没有使用编解码器，它是最有效的。但是，当配置的积极结果可以实现[S3选择]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3#using-s3_select-with-the-s3-source)。

## AVRO

[Apache Avro](helps streamline streaming data pipelines. It is most efficient when used with the [`avro` codec]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/s3#avro-codec) 内部`s3` 下沉。



---
layout: default
title: 常见的休息参数
nav_order: 93
redirect_from:
  - /opensearch/common-parameters/
---

# 常见的休息参数
**引入1.0**
{: .label .label-purple }

OpenSearch支持所有REST操作的以下参数：

## 人类-可读的输出

将输出单位转换为人类-可读值（例如，`1h` 1小时`1kb` 对于1,024个字节），添加`?human=true` 请求URL。

#### 示例请求

以下请求要求响应值在人类中-可读格式：

```json

GET <index_name>/_search?human=true
```

## 漂亮的结果

要以可读格式恢复JSON响应，请添加`?pretty=true` 请求URL。

#### 示例请求

以下请求需要以漂亮的JSON格式显示响应：

```json

GET <index_name>/_search?pretty=true
```

## 内容类型

要指定请求主体中内容的类型，请使用`Content-Type` 请求标题中的密钥名称。大多数操作都支持JSON，YAML和CBOR格式。

#### 示例请求

以下请求指定请求主体的JSON格式：

```json

curl -H "Content-type: application/json" -XGET localhost:9200/_scripts/<template_name>
```

## 在查询字符串中请求主体

如果客户库库不接受非请求机构-发布请求，使用`source` 查询字符串参数通过请求主体。另外，指定`source_content_type` 具有支持的媒体类型的参数，例如`application/json`。


#### 示例请求

以下请求搜索了文档`shakespeare` 特定字段和价值的索引：

```json

GET shakespeare/search?source={"query":{"exists":{"field":"speaker"}}}&source_content_type=application/json
```

## 堆栈跟踪

要在提出异常时在响应中包括错误堆栈跟踪，请添加`error_trace=true` 请求URL。

#### 示例请求

以下请求集`error_trace` 到`true` 因此响应返回例外-触发错误：

```json

GET <index_name>/_search?error_trace=true
```

## 过滤响应

为了减少响应尺寸`filter_path` 参数以过滤返回的字段。此参数需要一个逗号-分开的过滤器列表。它支持使用通配符匹配字段名称的任何字段或一部分。您也可以将字段排除在`-`。

#### 示例请求

以下请求指定过滤器以限制响应中返回的字段：

```json

GET _search?filter_path=<field_name>.*,-<field_name>
```


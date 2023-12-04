---
layout: default
title: 克隆索引
parent: 索引API
nav_order: 15
redirect_from:
  - /opensearch/rest-api/index-apis/clone/
---

# 克隆索引
**引入1.0**
{: .label .label-purple }

克隆索引API操作克隆现有读取中的所有数据-仅索引为新索引。新索引已经不存在。

## 例子

```json
PUT /sample-index1/_clone/cloned-index1
{
  "settings": {
    "index": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    }
  },
  "aliases": {
    "sample-alias1": {}
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST /<source-index>/_clone/<target-index>
PUT /<source-index>/_clone/<target-index>
```

## 索引命名限制

OpenSearch索引具有以下命名限制：

- 所有字母必须是小写。
- 索引名称不能以下划线开头（`_`）或连字符（`-`）。
- 索引名称不能包含空格，逗号或以下字符：

  `:`，`"`，`*`，`+`，`/`，`\`，`|`，`?`，`#`，`>`， 或者`<`

## URL参数

您的请求必须包括源索引和目标索引。所有其他克隆索引参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
＆lt;来源-索引＆gt;| 细绳| 克隆的源索引。
＆lt;目标-索引＆gt;| 细绳| 创建和添加克隆数据的索引。
wait_for_active_shards| 细绳| 在OpenSearch处理请求之前，必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置为全部或正整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则索引必须在两个其他节点上分布两个副本才能成功。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待请求返回多长时间。默认为`30s`。
wait_for_completion| 布尔| 设置为`false`，请求立即返回操作完成后。要监视操作状态，请使用[任务API]({{site.url}}{{site.baseurl}}/api-reference/tasks/) 随着请求返回任务ID。默认为`true`。
task_execution_timeout| 时间| 明确的任务执行超时。仅当将WAIT_FOR_COMPLETION设置为`false`。默认为`1h`。

## 请求身体

克隆索引API操作创建了一个新的目标索引，因此您可以指定任何[索引设置]({{site.url}}{{site.baseurl}}/im-plugin/index-settings/) 和[别名]({{site.url}}{{site.baseurl}}/opensearch/index-alias/) 应用于目标索引。

## 回复

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "cloned-index1"
}
```


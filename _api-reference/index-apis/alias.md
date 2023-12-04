---
layout: default
title: 别名
parent: 索引API
nav_order: 5
redirect_from: 
 - /opensearch/rest-api/alias/
 - /api-reference/alias/
---

# 别名
**引入1.0**
{: .label .label-purple }

别名是一个虚拟指针，您可以用来引用一个或多个索引。创建和更新别名是原子操作，因此您可以重新索引数据并指向别名，而无需任何停机时间。


## 例子

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies",
        "alias": "movies-alias1"
      }
    },
    {
      "remove": {
        "index": "old-index",
        "alias": "old-index-alias"
      }
    }
  ]
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST _aliases
```

## URL参数

所有别名参数都是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
cluster_manager_timeout| 时间| 等待群集管理器节点响应的时间。默认为`30s`。
暂停| 时间| 等待集群响应的时间。默认为`30s`。

## 请求身体

在您的请求主体中，您需要指定要采取的措施，别名名称以及要与别名相关联的索引。其他字段是可选的。

场地| 数据类型| 描述| 必需的
:--- | :--- | :--- | :---
动作| 大批| 您想在索引上执行的一组操作。有效选项是：`add`，`remove`， 和`remove_index`。您必须在数组中至少有一个动作。| 是的
添加| N/A。| 向指定索引添加一个别名。| 不
消除| N/A。| 从指定索引中删除别名。| 不
remove_index| N/A。| 删除索引。| 不
指数| 细绳| 您要与别名相关联的索引的名称。支持通配符表达式。| 是的，如果您不提供`indices` 体内的场。
指数| 大批| 您要与别名相关联的索引名称。| 是的，如果您不提供`index` 体内的场。
别名| 细绳| 别名的名称。| 是的，如果您不提供`aliases` 体内的场。
别名| 大批| 别名名称。| 是的，如果您不提供`alias` 体内的场。
筛选| 目的| 与别名一起使用的过滤器，因此别名指向索引的过滤部分。| 不
is_hidden| 布尔| 指定是否应从包括通配符表达式的结果中隐藏别名| 不
MUST_EXIST| 布尔| 指定是否必须存在删除的别名。| 不
is_write_index| 布尔| 指定索引是否应为写索引。一个别名一次只能有一个写索引。如果将写请求提交给链接到多个索引的别名，则OpenSearch仅在写索引上执行请求。| 不
路由| 细绳| 用于为特定操作分配自定义值。| 不
index_routing| 细绳| 仅为索引操作分配自定义值。| 不
search_routing| 细绳| 将自定义值分配给仅用于搜索操作的碎片。| 不

## 回复

```json
{
    "acknowledged": true
}
```

有关更多别名API操作，请参阅[索引别名]({{site.url}}{{site.baseurl}}/opensearch/index-alias/)


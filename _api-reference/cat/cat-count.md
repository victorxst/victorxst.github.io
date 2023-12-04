---
layout: default
title: CAT count
parent: CAT API
nav_order: 10
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-count/

---

# 猫数
**引入1.0**
{: .label .label-purple }

CAT计数操作列出了群集中的文档数量。

## 例子

```json
GET _cat/count?v
```
{% include copy-curl.html %}

要查看特定索引或别名中的文档数量，请在查询之后添加索引或别名名称：

```json
GET _cat/count/<index_or_alias>?v
```
{% include copy-curl.html %}

如果您想获取多个索引或别名的信息，请将索引或别名名称与逗号分开：

```json
GET _cat/count/index_or_alias_1,index_or_alias_2,index_or_alias_3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/count?v
GET _cat/count/<index>?v
```

## URL参数

所有CAT计数URL参数都是可选的。您可以指定任何[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)。


## 回复

以下响应显示总体文档数量为1625：

```json
epoch      | timestamp | count
1624237738 | 01:08:58  | 1625
```


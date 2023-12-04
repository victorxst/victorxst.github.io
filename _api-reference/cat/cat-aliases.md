---
layout: default
title: CAT 别名
parent: CAT API
redirect_from:
- /opensearch/rest-api/cat/cat-aliases/

nav_order: 1
has_children: false
---

# 猫别名
**引入1.0**
{: .label .label-purple }

CAT别名操作列出了别名与索引的映射，以及路由和过滤信息。

## 例子

```json
GET _cat/aliases?v
```
{% include copy-curl.html %}

要将信息限制为特定别名，请在查询之后添加别名名称：

```json
GET _cat/aliases/<alias>?v
```
{% include copy-curl.html %}

如果您想获取多个别名的信息，请将别名名称与逗号分开：

```json
GET _cat/aliases/alias1,alias2,alias3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/aliases/<alias>
GET _cat/aliases
```


## URL参数

所有CAT别名URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从主节点返回。默认值为false。
Expand_WildCard| 枚举| 将通配符表达式扩展到混凝土指数。将多个值与逗号相结合。支持的值是`all`，`open`，`closed`，`hidden`， 和`none`。默认为`open`。

## 回复

以下回应表明`alias1` 指的是`movies` 索引并具有配置的过滤器：

```json
alias   | index     | filter  | routing.index | routing.search  | is_write_index
alias1  | movies    |   *     |      -        |       -         |      -
.opensearch-dashboards | .opensearch-dashboards_1 |   -     |      -        |       -         |      -
```

要了解有关索引别名的更多信息，请参阅[索引别名]({{site.url}}{{site.baseurl}}/opensearch/index-alias)。


---
layout: default
title: CAT 模板
parent: CAT API

nav_order: 70
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-templates/
---

# 猫模板
**引入1.0**
{: .label .label-purple }

CAT模板操作列出了索引模板的名称，模式，订单编号和版本号。

## 例子

```
GET _cat/templates?v
```
{% include copy-curl.html %}

如果要获取特定模板或模式的信息：

```
GET _cat/templates/<template_name_or_pattern>
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/templates
```
{% include copy-curl.html %}

## URL参数

所有CAT模板URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从群集管理器节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。


## 回复

```
name | index_patterns order version composed_of
tenant_template | [opensearch-dashboards*] | 0  |    
```

要了解有关索引模板的更多信息，请参阅[索引模板]({{site.url}}{{site.baseurl}}/opensearch/index-templates)。


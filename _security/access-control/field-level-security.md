---
layout: default
title: 现场级安全
parent: 访问控制
nav_order: 90
redirect_from:
 - /security/access-control/field-level-security/
 - /security-plugin/access-control/field-level-security/
---

# 现场级安全

场地-级别的安全性使您可以控制用户可以看到的哪些文档字段。就像[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security/)，您可以在角色内通过索引来控制访问。

开始文档的最简单方法- 和字段-级别安全是开放式搜索仪表板，然后选择**安全**。然后选择**角色**，创建一个新角色，并审查**指数许可** 部分。

---

#### Table of contents
1. TOC
{:toc}


---

## 包括或排除字段

配置字段时，您有两个选择-级别安全性：包括或排除字段。如果包含字段，则用户在检索文档时只会看到 * *这些字段。例如，如果您包括`actors`，，，，`title`， 和`year` 字段，搜索结果可能看起来像这样：

```json
{
  "_index": "movies",
  "_source": {
    "year": 2013,
    "title": "Rush",
    "actors": [
      "Daniel Brühl",
      "Chris Hemsworth",
      "Olivia Wilde"
    ]
  }
}
```

如果您排除字段，用户会看到所有内容 *但是 *这些字段在检索文档时。例如，如果您排除了相同的字段，则相同的搜索结果可能看起来像：

```json
{
  "_index": "movies",
  "_source": {
    "directors": [
      "Ron Howard"
    ],
    "plot": "A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda.",
    "genres": [
      "Action",
      "Biography",
      "Drama",
      "Sport"
    ]
  }
}
```

您可以使用包含或排除实现相同的结果，因此请选择对您的用例有意义的选择。混合两者是没有意义的，也不支持。

您可以指定字段-使用OpenSearch仪表板的级别安全设置，`roles.yml`和其余的API。

- 排除字段`roles.yml` 或REST API，添加`~` 在字段名称之前。
- 字段名称支持通配符（`*`）。

  通配符对于排除 *子场 *特别有用。例如，如果索引具有字符串的文档（例如`{"title": "Thor"}`），OpenSearch创建`title` 类型字段`text`，但这也创造了一个`title.keyword` 类型子字段`keyword`。在此示例中，为了防止未经授权访问数据`title` 字段，您还必须排除`title.keyword` 子场。使用`title*` 匹配以开头的所有字段`title`。


### OpenSearch仪表板

1. 选择角色和**添加索引许可**。
1. 选择索引模式。
1. 在下面**现场级别的安全性**，使用滴-向下选择您的首选选项。然后指定一个或多个字段，然后按Enter。


### 角色

```yml
someonerole:
  cluster: []
  indices:
    movies:
      '*':
      - "READ"
      _fls_:
      - "~actors"
      - "~title"
      - "~year"
```

### REST API

看[创建角色]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-role)。


## 与多个角色的互动

如果将用户映射到多个角色，我们建议这些角色使用include *或 *排除每个索引的语句。安全插件评估字段-使用级别的安全设置`AND` 操作员，因此结合和排除语句可以导致行为既不正常。

例如，在`movies` 索引，如果包括`actors`，，，，`title`， 和`year` 在一个角色中，排除`actors`，，，，`title`， 和`genres` 在另一个角色中，然后将两个角色都映射到同一用户，搜索结果可能看起来像：

```json
{
  "_index": "movies",
  "_source": {
    "year": 2013,
    "directors": [
      "Ron Howard"
    ],
    "plot": "A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda."
  }
}
```


## 与文档的互动-级别的安全性

[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security/) 依靠OpenSearch查询，这意味着必须看到查询中的所有字段才能正常工作。如果您使用字段-与文档结合使用的级别安全性-级别的安全性，请确保您不限制对文档的字段的访问-级别安全用途。


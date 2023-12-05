---
layout: default
title: 场遮蔽
parent: 访问控制
nav_order: 95
redirect_from:
 - /security/access-control/field-masking/
 - /security-plugin/access-control/field-masking/
---

# 场遮蔽

如果您不想使用文档中删除字段[场地-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/field-level-security/)，您可以掩盖它们的价值观。当前，字段掩码仅适用于字符串-基于字段并用加密哈希替换字段的值。

现场蒙版与现场一起工作-相同的级别安全性-角色，per-索引基础。您可以允许某些角色在纯文本中查看敏感的字段，并为他人掩盖它们。带有蒙版字段的搜索结果看起来像这样：

```json
{
  "_index": "movies",
  "_source": {
    "year": 2013,
    "directors": [
      "Ron Howard"
    ],
    "title": "ca998e768dd2e6cdd84c77015feb29975f9f498a472743f159bec6f1f1db109e"
  }
}
```


## 放盐

您将盐设置为盐（用于哈希数据的随机字符串）`opensearch.yml`：

```yml
plugins.security.compliance.salt: abcdefghijklmnopqrstuvqxyz1234567890
```

财产| 描述
:--- | :---
`plugins.security.compliance.salt` | 生成哈希值时要使用的盐。必须至少32个字符。仅允许ASCII字符。选修的。

设置盐是可选的，但我们强烈建议它。


## 配置字段掩码

您使用OpenSearch仪表板配置字段掩码，`roles.yml`，或剩下的API。

### OpenSearch仪表板

1. 选择角色。
1. 选择索引许可。
1. 为了**匿名化**，指定一个或多个字段，然后按Enter。


### 角色

```yml
someonerole:
  cluster: []
  indices:
    movies:
      _masked_fields_:
      - "title"
      - "genres"
      '*':
      - "READ"
```


### REST API

看[创建角色]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-role)。


## （高级）使用替代哈希算法

默认情况下，安全插件使用Blake2B算法，但是您可以使用JVM提供的任何哈希算法。此列表通常包括MD5，SHA-1，莎-384和莎-512。

要指定其他算法，请在蒙版字段之后添加：

```yml
someonerole:
  cluster: []
  indices:
    movies:
      _masked_fields_:
      - "title::SHA-512"
      - "genres"
      '*':
      - "READ"
```


## （高级）模式-基于现场掩蔽

您可以使用一个或多个正则表达式和替换字符串来掩盖字段，而不是创建哈希。语法是`<field>::/<regular-expression>/::<replacement-string>`。如果您使用多个正则表达式，则结果将从从左到右传递，例如在外壳中的管道：

```yml
hr_employee:
  index_permissions:
    - index_patterns:
      - 'humanresources'
      allowed_actions:
        - ...
      masked_fields:
        - 'lastname::/.*/::*'
        - '*ip_source::/[0-9]{1,3}$/::XXX::/^[0-9]{1,3}/::***'
someonerole:
  cluster: []
  indices:
    movies:
      _masked_fields_:
      - "title::/./::*"
      - "genres::/^[a-zA-Z]{1,3}/::XXX::/[a-zA-Z]{1,3}$/::YYY"
      '*':
      - "READ"

```

这`title` 语句将字段中的每个字符更改为`*`，因此您仍然可以辨别蒙版字符串的长度。这`genres` 语句将字符串的前三个字符更改为`XXX` 最后三个字符`YYY`。


## 对审计记录的影响

读取历史记录功能使您可以跟踪文档中对敏感字段的访问。例如，您可能会跟踪对客户记录的电子邮件字段的访问。从读取历史记录中排除了对蒙版字段的访问，因为用户只看到了哈希值，而不是该字段的清晰文本值。


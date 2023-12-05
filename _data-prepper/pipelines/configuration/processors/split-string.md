---
layout: default
title: split_string
parent: 处理器
grand_parent: 管道
nav_order: 100
---

# split_string


这`split_string` 处理器使用划界字符将字段分为数组，为[突变字符串](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/mutate-string-processors#mutate-string-processors) 处理器。下表描述了您可以使用的选项来配置`split_string` 处理器。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
条目| 是的| 列表| 条目列表。有效值是`source`，`delimiter`， 和`delimiter_regex`。
来源| N/A。| N/A。| 拆分的钥匙。
定界符| 不| N/A。| 分离器角色负责拆分。不能与`delimiter_regex`。至少`delimiter` 或者`delimiter_regex` 必须定义。
delimiter_regex| 不| N/A。| 负责拆分的正则弦字符串。不能与`delimiter`。至少`delimiter` 或者`delimiter_regex` 必须定义。

<！---## 配置

内容将添加到本节中。

## 指标

内容将添加到本节中。---


---
layout: default
title: substitute_string
parent: 处理器
grand_parent: 管道
nav_order: 110
---

# 替代_STRING

这`substitute_string` 处理器将密钥的值与正则表达式匹配，并用替换字符串替换所有匹配项。`substitute_string` 是一个[突变字符串](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/mutate-string-processors#mutate-string-processors) 处理器。

## 配置

下表描述了您可以使用的选项来配置`substitue_string` 处理器。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
条目| 是的| 列表| 条目列表。有效值是`source`，`from`， 和`to`。
来源| N/A。| N/A。| 修改的关键。
从| N/A。| N/A。| 将要替换的正则弦字符串。特殊的正则角色，例如`[` 和`]( must be escaped using `\\` when using double quotes and `\ ` when using single quotes. See [Java Patterns](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/Pattern.html) for more information.
to | N/A | N/A | The String to be substituted for each match of `从`。

<！---## 配置

内容将添加到本节中。

## 指标

内容将添加到本节中。---


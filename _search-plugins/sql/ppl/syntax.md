---
layout: default
title: 句法
parent: PPL &ndash; 管道处理语言
grand_parent: SQL和PPL
nav_order: 1
---

# ppl语法

每个ppl查询都以`search` 命令。它指定了搜索和检索文档的索引。随后的命令可以按任何顺序遵循。

现在，`PPL` 仅支持一个`search` 命令，可以省略以简化查询。
{ ： 。笔记}

## 句法

```sql
search source=<index> [boolean-expression]
source=<index> [boolean-expression]
```

场地| 描述| 必需的
:--- | :--- |：---
`search` | 指定搜索关键字。| 是的
`index` | 指定索引要查询的索引。| 不
`bool-expression` | 指定评估布尔值的表达式。| 不

## 例子

**示例1：通过帐户索引搜索**

在下面的示例中，`search` 命令是指`accounts` 索引作为源并使用`fields` 和`where` 条件的命令：

```sql
search source=accounts
| where age > 18
| fields firstname, lastname
```

在以下示例中，角括号`< >` 封闭所需的参数和方括号`[ ]` 包含可选参数。
{: .note}


**示例2：获取所有文档**

从`accounts` 索引，将其指定为`source`：

```sql
search source=accounts;
```

| 帐号| 名| 地址| 平衡| 性别| 城市| 雇主| 状态| 年龄| 电子邮件| 姓|
:--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---
| 1| 琥珀色| 880 Holmes Lane| 39225| m| 布罗根| Pyrami| il| 32| amberduke@pyrami.com| 公爵
| 6| 哈蒂| 布里斯托尔街671号| 5686| m| 但丁| 网络| TN| 36| hattiebond@netagy.com| 纽带
| 13| Nanette| 麦迪逊街789号| 32838| F| Nogal| 裁员| VA| 28| 无效的| 贝茨
| 18| 戴尔| 467 Hutchinson Court| 4180| m| 奥里克| 无效的| MD| 33| daleadams@boink.com| 亚当斯

**示例3：获取与条件匹配的文档**

从`accounts` 有索引`account_number` 等于1或`gender` 作为`F`，使用以下查询：

```sql
search source=accounts account_number=1 or gender=\"F\";
```

| 帐号| 名| 地址| 平衡| 性别| 城市| 雇主| 状态| 年龄| 电子邮件| 姓|
:--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---
| 1| 琥珀色| 880 Holmes Lane| 39225| m| 布罗根| Pyrami| il| 32| amberduke@pyrami.com| 公爵|
| 13| Nanette| 麦迪逊街789号| 32838| F| Nogal| 裁员| VA| 28| 无效的| 贝茨|


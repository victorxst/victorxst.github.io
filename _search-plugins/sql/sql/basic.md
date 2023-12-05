---
layout: default
title: 基本查询
parent: SQL
grand_parent: SQL和PPL
nav_order: 5
Redirect_from:
  - /search-plugins/sql/basic/
---


# 基本查询

使用`SELECT` 条款，以及`FROM`，，，，`WHERE`，，，，`GROUP BY`，，，，`HAVING`，，，，`ORDER BY`， 和`LIMIT` 搜索和汇总数据。

在这些条款中，`SELECT` 和`FROM` 需要，因为他们指定要检索哪些字段以及从哪些字段中检索它们。所有其他条款都是可选的。根据您的需求使用它们。

### 句法

用于搜索和汇总数据的完整语法如下：

```sql
SELECT [DISTINCT] (* | expression) [[AS] alias] [, ...]
FROM index_name
[WHERE predicates]
[GROUP BY expression [, ...]
 [HAVING predicates]]
[ORDER BY expression [IS [NOT] NULL] [ASC | DESC] [, ...]]
[LIMIT [offset, ] size]
```

### 基本面

除了SQL的预定义关键字外，最基本的元素是字面和标识符。
文字是数字，字符串，日期或布尔常数。标识符是OpenSearch索引或字段名称。
使用算术运算符和SQL功能，使用文字和标识符来构建复杂的表达式。

规则`expressionAtom`：

![表达状态]({{site.url}}{{site.baseurl}}/images/expressionAtom.png)

依次可以将表达式与逻辑运算符合并为谓词。在`WHERE` 和`HAVING` 条款以特定条件过滤数据。

规则`expression`：

![表达]({{site.url}}{{site.baseurl}}/images/expression.png)

规则`predicate`：

![表达]({{site.url}}{{site.baseurl}}/images/predicate.png)

### 执行顺序

这些SQL条款的执行顺序与它们的外观不同：

```sql
FROM index
 WHERE predicates
  GROUP BY expressions
   HAVING predicates
    SELECT expressions
     ORDER BY expressions
      LIMIT size
```

## 选择

指定要检索的字段。

### 句法

规则`selectElements`：

![选择点]({{site.url}}{{site.baseurl}}/images/selectElements.png)

规则`selectElement`：

![选择点]({{site.url}}{{site.baseurl}}/images/selectElement.png)

*示例1*：使用`*` 在索引中检索所有字段：

```sql
SELECT *
FROM accounts
```

| 帐号| 名| 性别| 城市| 平衡| 雇主| 状态| 电子邮件| 地址| 姓| 年龄
| ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：---
| 1| 琥珀色| m| 布罗根| 39225| Pyrami| il| amberduke@pyrami.com| 880 Holmes Lane| 公爵| 32
| 16| 哈蒂| m| 但丁| 5686| 网络| TN| hattiebond@netagy.com| 布里斯托尔街671号| 纽带| 36
| 13| Nanette| F| Nogal| 32838| 裁员| VA| nanettebates@quility.com| 麦迪逊街789号| 贝茨| 28
| 18| 戴尔| m| 奥里克| 4180|  | MD| daleadams@boink.com| 467 Hutchinson Court| 亚当斯| 33

*示例2*：使用字段名称仅检索特定字段：

```sql
SELECT firstname, lastname
FROM accounts
```

| 名| 姓
| ：--- | ：---
| 琥珀色| 公爵
| 哈蒂| 纽带
| Nanette| 贝茨
| 戴尔| 亚当斯

*示例3*：使用字段别名而不是字段名称。字段别名用于使字段名称更具可读性：

```sql
SELECT account_number AS num
FROM accounts
```

| num
：---
| 1
| 6
| 13
| 18

*示例4*：使用`DISTINCT` 条款仅返回唯一的字段值。您可以指定一个或多个字段名称：

```sql
SELECT DISTINCT age
FROM accounts
```

| 年龄
：---
| 28
| 32
| 33
| 36

## 从

指定要搜索的索引。
您可以在`FROM` 条款。

### 句法

规则`tableName`：

![tablename]({{site.url}}{{site.baseurl}}/images/tableName.png)

*示例1*：使用索引别名跨索引查询。要了解索引别名，请参阅[索引别名]({{site.url}}{{site.baseurl}}/opensearch/index-alias/)。
在此示例查询中，`acc` 是一个别名`accounts` 指数：

```sql
SELECT account_number, accounts.age
FROM accounts
```

或者

```sql
SELECT account_number, acc.age
FROM accounts acc
```

| 帐号| 年龄
| ：--- | ：---
| 1| 32
| 6| 36
| 13| 28
| 18| 33

*示例2*：使用索引模式查询匹配特定模式的索引：

```sql
SELECT account_number
FROM account*
```

| 帐号
：---
| 1
| 6
| 13
| 18

## 在哪里

指定过滤结果的条件。

| 操作员| 行为
：--- | ：---
`=` | 等于。
`<>` | 不等于。
`>` | 比...更棒。
`<` | 少于。
`>=` | 大于或等于。
`<=` | 小于或等于。
`IN` | 指定倍数`OR` 操作员。
`BETWEEN` | 类似于范围查询。有关范围查询的更多信息，请参阅[范围查询]({{site.url}}{{site.baseurl}}/query-dsl/term/range/)。
`LIKE` | 用于完整-文字搜索。有关完整的更多信息-文字查询，请参阅[满的-文本查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/index/)。
`IS NULL` | 检查字段值是否为`NULL`。
`IS NOT NULL` | 检查字段值是否为`NOT NULL`。

结合比较操作员（`=`，，，，`<>`，，，，`>`，，，，`>=`，，，，`<`，，，，`<=`）与布尔运营商`NOT`，，，，`AND`， 或者`OR` 建立更复杂的表情。

*示例1*：使用比较操作员进行数字，字符串或日期：

```sql
SELECT account_number
FROM accounts
WHERE account_number = 1
```

| 帐号
| ：---
| 1

*示例2*：OpenSearch允许使用灵活的模式，因此索引中的文档可能具有不同的字段。使用`IS NULL` 或者`IS NOT NULL` 仅检索缺失字段或现有字段。OpenSearch不会区分缺失字段和明确设置为`NULL`：

```sql
SELECT account_number, employer
FROM accounts
WHERE employer IS NULL
```

| 帐号| 雇主
| ：--- | ：---
| 18|

*示例3*：删除满足谓词的文档`WHERE` 条款：

```sql
DELETE FROM accounts
WHERE age > 30
```

## 通过...分组

将具有相同字段值的组文档分为存储桶。

*示例1*：组成的组：

```sql
SELECT age
FROM accounts
GROUP BY age
```

| ID| 年龄
：--- | ：---
0| 28
1| 32
2| 33
3| 36

*示例2*：组性别名的组：

```sql
SELECT account_number AS num
FROM accounts
GROUP BY num
```

| ID| num
：--- | ：---
0| 1
1| 6
2| 13
3| 18

*示例4*：使用标量功能`GROUP BY` 条款：

```sql
SELECT ABS(age) AS a
FROM accounts
GROUP BY ABS(age)
```

| ID| A
：--- | ：---
0| 28.0
1| 32.0
2| 33.0
3| 36.0

## 有

使用`HAVING` 基于聚合函数在每个存储桶内的子句（子句）（`COUNT`，，，，`AVG`，，，，`SUM`，，，，`MIN`， 和`MAX`）。
这`HAVING` 子句过滤来自`GROUP BY` 条款：

*示例1*：

```sql
SELECT age, MAX(balance)
FROM accounts
GROUP BY age HAVING MIN(balance) > 10000
```

| ID| 年龄| 最大（平衡）
：--- | ：---
0| 28| 32838
1| 32| 39225

## 订购

使用`ORDER BY` 条款将结果分类为您所需的订单。

*示例1*：使用`ORDER BY` 通过上升或降序排序。除了常规字段名称，使用`ordinal`，，，，`alias`， 或者`scalar` 支持功能：

```sql
SELECT account_number
FROM accounts
ORDER BY account_number DESC
```

| 帐号
| ：---
| 18
| 13
| 6
| 1

*示例2*：指定是否要将具有缺失字段的文档放在结果的开始或结束时。OpenSearch的默认行为是返回末尾的nulls或缺少字段。将它们推到非-nulls，使用`IS NOT NULL` 操作员：

```sql
SELECT employer
FROM accounts
ORDER BY employer IS NOT NULL
```

| 雇主
| ：---
||
| 网络
| Pyrami
| 裁员

## 限制

指定要检索的最大文档数量。用于防止将大量数据获取到内存中。

*示例1*：如果您通过一个参数，则将其映射到`size` OpenSearch和`from` 参数设置为0。

```sql
SELECT account_number
FROM accounts
ORDER BY account_number LIMIT 1
```

| 帐号
| ：---
| 1

*示例2*：如果您通过两个参数，则第一个映射到`from` 参数，第二个`size` opensearch中的参数。您可以将其用于简单的小索引，因为它对于大索引效率低下。
使用`ORDER BY` 确保页面之间相同的顺序：

```sql
SELECT account_number
FROM accounts
ORDER BY account_number LIMIT 1, 1
```

| 帐号
| ：---
| 6


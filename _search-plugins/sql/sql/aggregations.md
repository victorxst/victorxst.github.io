---
layout: default
title: 聚合功能
parent: SQL
grand_parent: SQL和PPL
nav_order: 11
Redirect_from:
  - /search-plugins/sql/aggregations/
---

# 聚合功能

汇总功能在由`GROUP BY` 条款。在没有`GROUP BY` 子句，汇总函数在结果集的所有元素上运行。您可以在`GROUP BY`，，，，`SELECT`， 和`HAVING` 条款。

OpenSearch支持以下汇总功能。

功能| 描述
：--- | ：---
`AVG` | 返回结果的平均值。
`COUNT` | 返回结果数。
`SUM` | 返回结果的总和。
`MIN` | 返回结果的最小值。
`MAX` | 返回结果的最大值。
`VAR_POP` 或者`VARIANCE` | 返回丢弃零值后结果的种群差异。返回0时只有一行结果。
`VAR_SAMP` | 返回丢弃nulls后结果的样本方差。返回null时只有一行结果。
`STD` 或者`STDDEV` | 返回结果的样本标准偏差。返回0时只有一行结果。
`STDDEV_POP` | 返回结果的人口标准偏差。返回0时只有一行结果。
`STDDEV_SAMP` | 返回结果的样本标准偏差。返回null时只有一行结果。

下面的示例参考`employees` 桌子。您可以通过使用批量索引操作将以下文档索引到OpenSearch中来尝试示例：

```json
PUT employees/_bulk?refresh
{"index":{"_id":"1"}}
{"employee_id": 1, "department":1, "firstname":"Amber", "lastname":"Duke", "sales":1356, "sale_date":"2020-01-23"}
{"index":{"_id":"2"}}
{"employee_id": 1, "department":1, "firstname":"Amber", "lastname":"Duke", "sales":39224, "sale_date":"2021-01-06"}
{"index":{"_id":"6"}}
{"employee_id":6, "department":1, "firstname":"Hattie", "lastname":"Bond", "sales":5686, "sale_date":"2021-06-07"}
{"index":{"_id":"7"}}
{"employee_id":6, "department":1, "firstname":"Hattie", "lastname":"Bond", "sales":12432, "sale_date":"2022-05-18"}
{"index":{"_id":"13"}}
{"employee_id":13,"department":2, "firstname":"Nanette", "lastname":"Bates", "sales":32838, "sale_date":"2022-04-11"}
{"index":{"_id":"18"}}
{"employee_id":18,"department":2, "firstname":"Dale", "lastname":"Adams", "sales":4180, "sale_date":"2022-11-05"}
```

## 通过...分组

这`GROUP BY` 子句定义结果集的子集。聚合功能在这些子集上运行，并为每个子集返回一个结果行。

您可以在`GROUP BY` 条款。

### 通过在组中使用标识符

您可以指定字段名称（列名）以在`GROUP BY` 条款。例如，以下查询返回部门编号和每个部门的总销售额：
```sql
SELECT department, sum(sales) 
FROM employees 
GROUP BY department;
```

| 部门| 总和（销售）
：--- | ：---
1| 58700|
2| 37018|

### 通过组中的序数

您可以指定要在`GROUP BY` 条款。列号由列位置确定`SELECT` 条款。例如，以下查询等同于上面的查询。它返回部门号码和每个部门的总销售额。它按结果集的第一列将结果分组为`department`：

```sql
SELECT department, sum(sales) 
FROM employees 
GROUP BY 1;
```

| 部门| 总和（销售）
：--- | ：---
1| 58700|
2| 37018|

### 通过在组中使用表达式

您可以在`GROUP BY` 条款。例如，以下查询返回每年的平均销售额：

```sql
SELECT year(sale_date), avg(sales) 
FROM employees 
GROUP BY year(sale_date);
```

| 年（start_date）| AVG（销售）
：--- | ：---
| 2020| 1356.0|
| 2021| 22455.0|
| 2022| 16484.0|

## 选择

您可以在`SELECT` 子句直接或作为较大表达式的一部分。此外，您可以将表达式用作汇总函数的参数。

### 直接在SELECT中使用骨料表达式

以下查询返回每个部门的平均销售额：

```sql
SELECT department, avg(sales) 
FROM employees 
GROUP BY department;
```

| 部门| AVG（销售）
：--- | ：---
1| 14675.0|
2| 18509.0|

### 将骨料表达式用作选择中较大表达式的一部分

以下查询计算了每个部门员工的平均佣金，为平均销售额的5％：

```sql
SELECT department, avg(sales) * 0.05 as avg_commission 
FROM employees 
GROUP BY department;
```

| 部门| avg_commission
：--- | ：---
1| 733.75|
2| 925.45|

### 将表达式作为参数来汇总函数

以下查询计算每个部门的平均佣金金额。首先，它计算每个的佣金金额`sales` 价值为5％`sales`。然后，它决定了所有佣金价值的平均值：

```sql
SELECT department, avg(sales * 0.05) as avg_commission 
FROM employees 
GROUP BY department;
```

| 部门| avg_commission
：--- | ：---
1| 733.75|
2| 925.45|

### 数数

这`COUNT` 功能接受参数，例如`*`，或文字，例如`1`。
下表描述了各种形式的方式`COUNT` 功能运行。

| 功能类型| 描述
`COUNT(field)` | 计算给定字段（或表达式）的值不为空的行数。
`COUNT(*)` | 计算表中的行总数。
`COUNT(1)` （与...一样`COUNT(*)`）| 计数任何非-零字面。

例如，以下查询返回每年的销售计数：

```sql
SELECT year(sale_date), count(sales) 
FROM employees 
GROUP BY year(sale_date);
```

| 年（sale_date）| 计数（销售）
：--- | ：---
2020| 1
2021| 2
2022| 3

## 有

两个都`WHERE` 和`HAVING` 用于过滤结果。这`WHERE` 过滤器在`GROUP BY` 阶段，因此您不能在`WHERE` 条款。但是，您可以使用`WHERE` 子句限制然后应用聚集的行。

这`HAVING` 过滤器在`GROUP BY` 阶段，因此您可以使用`HAVING` 条款以限制结果中包含的组。

### 与小组合作

您可以使用汇总表达式或在A中定义的别名`SELECT` 条款`HAVING` 健康）状况。

以下查询在`HAVING` 条款。它返回每位进行多个销售的员工的销售量：

```sql
SELECT employee_id, count(sales)
FROM employees
GROUP BY employee_id
HAVING count(sales) > 1;
```

| 员工ID| 计数（销售）
：--- | ：---
1| 2|
6| 2

一个集合`HAVING` 条款不必与一个集合相同`SELECT` 列表。以下查询使用`count` 功能在`HAVING` 条款但是`sum` 功能在`SELECT` 条款。它返回每位进行多个销售的员工的总销售额：

```sql
SELECT employee_id, sum(sales)
FROM employees
GROUP BY employee_id
HAVING count(sales) > 1;
```

| 员工ID| 总和（销售）
：--- | ：---
1| 40580|
6| 18120

作为SQL标准的扩展，您不仅限于仅使用标识符`GROUP BY` 条款。以下查询在`GROUP BY` 子句，等效于先前的查询：

```sql
SELECT employee_id as id, sum(sales)
FROM employees
GROUP BY id
HAVING count(sales) > 1;
```

| ID| 总和（销售）
：--- | ：---
1| 40580|
6| 18120

您也可以使用别名在`HAVING` 条款。以下查询返回销售额超过$ 40,000的每个部门的总销售额：

```sql
SELECT department, sum(sales) as total
FROM employees
GROUP BY department
HAVING total > 40000;
```

| 部门| 全部的
：--- | ：---
1| 58700|

如果标识符是模棱两可的（例如，同时将其作为一个`SELECT` 别名和索引字段），偏爱别名。在下面的查询中，标识符被替换为在`SELECT` 条款：

```sql
SELECT department, sum(sales) as sales
FROM employees
GROUP BY department
HAVING sales > 40000;
```

| 部门| 销售量
：--- | ：---
1| 58700|

### 没有小组

您可以使用`HAVING` 子句没有一个`GROUP BY` 条款。在这种情况下，整个数据集应视为一组。以下查询将返回`True` 如果有多个值`department` 柱子：

```sql
SELECT 'True' as more_than_one_department FROM employees HAVING min(department) < max(department);
```

| more_than_one_department|
：--- |
真的|

如果员工表中的所有员工都属于同一部门，则结果将包含零行：

| more_than_one_department
：--- |
 |


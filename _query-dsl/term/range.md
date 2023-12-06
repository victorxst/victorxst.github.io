---
layout: default
title: 范围
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 50
---

# 范围查询

您可以在字段中搜索具有`range` 询问。

搜索文档`line_id` 值是> = 10和<= 20，使用以下请求：

```json
GET shakespeare/_search
{
  "query": {
    "range": {
      "line_id": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```
{% include copy-curl.html %}

## 操作员

范围查询中的字段参数接受以下可选运算符参数：

- `gte`：大于或等于
- `gt`： 比...更棒
- `lte`：小于或等于
- `lt`： 少于

## 日期字段

您可以在包含日期的字段上使用范围查询。例如，假设您有一个`products` 索引，您想查找2019年添加的所有产品：

```json
GET products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2019/01/01",
        "lte": "2019/12/31"
      }
    }
  }
}
```
{% include copy-curl.html %}

有关支持日期格式的更多信息，请参见[格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/#formats)。

### 格式

要在查询中使用该字段映射格式以外的日期格式，请在`format` 场地。

例如，如果`products` 索引映射`created` 字段为`strict_date_optional_time`，您可以为查询日期指定其他格式，如下所示：

```json
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01/01/2022",
        "lte": "31/12/2022",
        "format":"dd/MM/yyyy"
      }
    }
  }
}
```
{% include copy-curl.html %}

### 缺少日期组件

OpenSearch填充缺少的日期组件，具有以下值：

- `MONTH_OF_YEAR`：`01`
- `DAY_OF_MONTH`：`01`
- `HOUR_OF_DAY`：`23`
- `MINUTE_OF_HOUR`：`59`
- `SECOND_OF_MINUTE`：`59`
- `NANO_OF_SECOND`：`999_999_999`

如果不丢失一年，则不会填充它。

例如，请考虑仅在开始日期中指定年份的以下请求：

```json
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2022",
        "lte": "2022-12-31"
      }
    }
  }
}
```
{% include copy-curl.html %}

开始日期用默认值填充，因此`gte` 使用的参数为`2022-01-01T23:59:59.999999999Z`。

### 相对日期

您可以使用[日期数学]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/#date-math)。

要从指定日期减去1年和1天，请使用以下查询：

```json
GET products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2019/01/01||-1y-1d"
      }
    }
  }
}
```
{% include copy-curl.html %}

在上一个示例中，`2019/01/01` 是日期数学的锚点日期（起点）。两个管道字符之后（`||`），您指定相对于锚点日期的数学表达式。在此示例中，您正在减去1年（`-1y`）和1天（`-1d`）。

您也可以通过在日期或时间单元中添加前向斜杠来解决日期。

要查找去年添加的产品，每月都可以使用以下查询：

```json
GET products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now-1y/M"
      }
    }
  }
}
```
{% include copy-curl.html %}

关键字`now` 指当前日期和时间。
{:.tip}

### 圆形相对日期

下表指定了相对日期的圆形方式。

范围| 舍入规则| 示例：值`2022-05-18||/M` 被舍入
:--- | :--- | :---
`gt` | 圆形到不在舍入间隔的第一个毫秒。| `2022-06-01T00:00:00.000`
`gte` | 到达第一毫秒。| `2022-05-01T00:00:00.000`
`lt` | 在圆形日期之前圆形到最后一个毫秒。| `2022-04-30T23:59:59.999`
`lte` | 在圆形间隔内到达最后一个毫秒。| `2022-05-31T23:59:59.999`

### 时区

默认情况下，假定日期在[协调的通用时间（UTC）](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)。如果指定`time_zone` 查询中的参数，提供的日期值将转换为UTC。您可以指定`time_zone` 参数为a[UTC偏移](https://en.wikipedia.org/wiki/UTC_offset)， 例如`-04:00`，或一个[IANA时区ID](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)， 例如`America/New_York`。例如，以下查询指定`gte` 查询中提供的日期是`-04:00` 时区：

```json
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "time_zone": "-04:00",        
        "gte": "2022-04-17T06:00:00" 
      }
    }
  }
}
```
{% include copy-curl.html %}

这`gte` 前面查询中的参数转换为`2022-04-17T10:00:00 UTC`，这是UTC等效的`2022-04-17T06:00:00-04:00`。

这`time_zone` 参数不影响`now` 价值是因为`now` 始终对应于UTC中的当前系统时间。
{: .note}

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "range": {
      "<field>": {
        "gt": 10,
        ... 
      }
    }
  }
}
```
{% include copy-curl.html %}


此外[操作员](#operators)，您可以为`<field>`。

范围| 数据类型| 描述
:--- | :--- | :---
`format` | 细绳| A[格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/#formats) 对于此查询中的日期。默认值是字段的映射格式。
`relation` | 细绳| 指示范围查询如何匹配值[`range`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/range/) 字段。有效值是：<br>- `INTERSECTS` （默认）：匹配其文档`range` 现场值与查询中提供的范围相交。<br>- `CONTAINS`：匹配文档`range` 字段值包含查询中提供的整个范围。<br>- `WITHIN`：匹配文档`range` 现场值完全在查询中提供的范围内。
`boost` | 漂浮的-观点| 通过给定的乘数增强查询。对于包含多个查询的搜索很有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。
`time_zone` | 细绳| 用于转换的时区[`date`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/) 在查询中的UTC值。有效值是ISO 8601[UTC偏移](https://en.wikipedia.org/wiki/List_of_UTC_offsets) 和[IANA时区ID](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。有关更多信息，请参阅[时区](#time-zone)。

如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，范围查询[`text`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text/) 和[`keyword`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/keyword/) 字段不运行。
{: .important}


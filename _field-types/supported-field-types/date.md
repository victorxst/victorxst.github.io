---
layout: default
title: 日期
nav_order: 25
has_children: false
parent: 日期字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/date/
  - /field-types/date/
---

# 日期字段类型

OpenSearch中的日期可以表示为以下一个：

- 由于时代以来，与毫秒相对应的长值（该值必须非-消极的）。日期在内部存储在此形式中。
- 格式的字符串。
- 自时代以来与秒相对应的整数值（该值必须是非-消极的）。

为了表示日期范围，有一个日期[范围字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/range/)。
{: .note }

## 例子

使用日期字段和两个日期格式创建映射：

```json
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "release_date" : {
        "type" : "date",
        "format" : "strict_date_optional_time||epoch_millis"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了由日期字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。默认为`false`。
`format` | 解析日期的格式。默认为`strict_date_time_no_millis||strict_date_optional_time||epoch_millis`。
`ignore_malformed` | 布尔值指定忽略畸形值而不引发异常的值。默认为`false`。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
`locale` | 一个地区- 和语言-代表日期的特定方式。默认为[`ROOT`](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT) （一个地区- 和语言-中立的地方）。
`meta` | 接受该领域的元数据。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。

## 格式

OpenSearch已建立-以日期格式，但您也可以创建自己的自定义格式。您可以指定多个日期格式，分开`||`。

## 默认格式

从OpenSearch 2.12开始，默认日期格式为`strict_date_time_no_millis||strict_date_optional_time||epoch_millis`。将默认格式恢复回到`strict_date_optional_time||epoch_millis` （OpenSearch 2.11及更早的默认格式），设置`opensearch.experimental.optimization.datetime_formatter_caching.enabled` 功能标志到`false`。有关启用和禁用功能标志的更多信息，请参阅[启用实验特征]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/experimental/)。

## 建造-以格式

大多数日期格式都有`strict_` 对准。当格式以`strict_`，该日期必须具有格式中指定的正确数字数字。例如，如果将格式设置为`strict_year_month_day` （（"yyyy-MM-dd"），每个月和一天都必须是两个-数字数字。所以，"2020-06-09" 有效，而"2020-6-9" 是无效的。

Epoch于1970年1月1日定义为00:00:00 UTC。
{: .note }

Y：年<br>
Y：[星期-基于一年](https://en.wikipedia.org/wiki/ISO_8601#Week_dates)<br>
m：月<br>
W：序数[一年](https://en.wikipedia.org/wiki/ISO_8601#Week_dates) 从01到53 <br>
D：天<br>
D：一年中的顺序日期从001到365（leap年366）<br>
E：一周中的顺序日1（星期一）到7（星期日）<br>
H：0到23的小时<br>
M：分钟<br>
S：第二<br>
S：一秒钟的部分<br>
Z：时区偏移（例如，+0400；-0400;-04:00）<br>
{: .note }

### 数字日期格式

格式名称和描述| 例子
:--- | :---
`epoch_millis` <br>自时代以来的毫秒数。最低是-2 <sup> 63 </sup>。最大值为2 <sup> 63 </sup>＆减去;1。| 1553391286000
`epoch_second` <br>自时代以来的秒数。最低是-2 <sup> 63 </sup>＆Divide;1000.最大值IS（2 <sup> 63 </sup>＆minus; 1）＆Divide;1000。| 1553391286

### 基本日期格式

基本日期格式的组成部分不会被定界符分开。例如，"20190323"。

格式名称和描述| 模式和示例
:--- | :---
**日期**| 
`basic_date_time` <br>一个基本日期和时间分开`T`。| "yyyyMMdd`T`HHmmss.SSSZ"<br>"20190323T213446.123-04:00"
`basic_date_time_no_millis` <br>基本日期和时间没有毫秒`T`。| "yyyyMMdd`T`HHmmssZ"<br>"20190323T213446-04:00"
`basic_date` <br>与四个约会-数字一年，两个-数字月份和两个-数字日。| "yyyyMMdd"<br>"20190323" 
**时代** |
`basic_time` <br>与两个时间-数字小时，两个-数分钟，两个-数字第二，三-数字毫秒和时区偏移。|"HHmmss.SSSZ" <br>"213446.123-04:00"
`basic_time_no_millis` <br>一个没有毫秒的基本时间。| "HHmmssZ" <br>"213446-04:00"
**T次** | 
`basic_t_time` <br>一个基本时间之前`T`。| "`T`HHmmss.SSSZ" <br>"T213446.123-04:00"
`basic_t_time_no_millis` <br>一个没有毫秒的基本时间，之前`T`。| "`T`HHmmssZ" <br>"T213446-04:00"
**序数日期** |
`basic_ordinal_date_time` <br>一个完整的序数日期和时间。| "yyyyDDD`T`HHmmss.SSSZ"<br>"2019082T213446.123-04:00"
`basic_ordinal_date_time_no_millis` <br>无毫秒的完整序数日期和时间。| "yyyyDDD`T`HHmmssZ"<br>"2019082T213446-04:00"
`basic_ordinal_date` <br>与四个约会-数字年份和三个-一年中的数字序日。| "yyyyDDD" <br>"2019082"
**星期-基于日期** | 
`basic_week_date_time` <br>`strict_basic_week_date_time` <br>整整一周-基于日期和时间分开`T`。| "YYYY`W`wwe`T`HHmmss.SSSZ" <br>"2019W126213446.123-04:00"
`basic_week_date_time_no_millis` <br>`strict_basic_week_date_time_no_millis` <br>基本的一周-基于年的日期和时间，没有毫秒，分开`T`。| "YYYY`W`wwe`T`HHmmssZ" <br>"2019W126213446-04:00"
`basic_week_date` <br>`strict_basic_week_date` <br>整整一周-基于四个的日期-数字周-基于一年，两个-一年中的数字序数周，一个-一周中的数字序日日相隔`W`。| "YYYY`W`wwe" <br>"2019W126"

### 全约日期格式

全约日期格式的组件由`-` 定界符日期和`:` 时间的定界符。例如，"2019-03-23T21:34"。

格式名称和描述| 模式和示例
:--- | :---
**日期** |
`date_optional_time`<br>`strict_date_optional_time` <br>通用的全日期和时间。需要年。一个月，一天和时间是可选的。时间从日期分开`T`。| 多种模式。<br>"2019-03-23T21:34:46.123456789-04:00" <br>"2019-03-23T21:34:46" <br>"2019-03-23T21:34" <br>"2019"
`strict_date_optional_time_nanos` <br>通用的全日期和时间。需要年。一个月，一天和时间是可选的。如果指定时间，则必须包含小时，分钟和秒，但是一秒钟的部分是可选的。一秒钟的一秒钟是长达九位数，并且具有纳秒分辨率。时间从日期分开`T`。| 多种模式。<br>"2019-03-23T21:34:46.123456789-04:00" <br>"2019-03-23T21:34:46" <br>"2019" 
`date_time` <br>`strict_date_time` <br>由`T`。| "yyyy-MM-dd`T`HH:mm:ss.SSSZ" <br>"2019-03-23T21:34:46.123-04:00"
`date_time_no_millis` <br>`strict_date_time_no_millis` <br>全约日期和时间没有毫秒`T`。| "yyyy-MM-dd'T'HH:mm:ssZ" <br>"2019-03-23T21:34:46-04:00" 
`date_hour_minute_second_fraction` <br>`strict_date_hour_minute_second_fraction` <br>一个全约会，两个-数字小时，两个-数分钟，两个-数字第二，一个- 到九-一秒钟分开的数字分数`T`。| "yyyy-MM-dd`T`HH:mm:ss.SSSSSSSSS"<br>"2019-03-23T21:34:46.123456789" <br>"2019-03-23T21:34:46.1"
`date_hour_minute_second_millis` <br>`strict_date_hour_minute_second_millis` <br>一个全约会，两个-数字小时，两个-数分钟，两个-数字第二，三个-数字毫秒`T`。| "yyyy-MM-dd`T`HH:mm:ss.SSS" <br>"2019-03-23T21:34:46.123" 
`date_hour_minute_second` <br>`strict_date_hour_minute_second` <br>一个全约会，两个-数字小时，两个-数字分钟和两个-数字第二分开`T`。| "yyyy-MM-dd`T`HH:mm:ss"<br>"2019-03-23T21:34:46" 
`date_hour_minute` <br>`strict_date_hour_minute` <br>一个全约会，两个-数字小时和两个-数字分钟。| "yyyy-MM-dd`T`HH:mm" <br>"2019-03-23T21:34"
`date_hour` <br>`strict_date_hour` <br>全约日期和两个-数字小时，分开`T`。| "yyyy-MM-dd`T`HH" <br>"2019-03-23T21" 
`date` <br>`strict_date` <br>四-数字一年，两个-数字月份和两个-数字日。| "yyyy-MM-dd" <br>"2019-03-23" 
`year_month_day` <br>`strict_year_month_day` <br>四-数字一年，两个-数字月份和两个-数字日。| "yyyy-MM-dd" <br>"2019-03-23" 
`year_month` <br>`strict_year_month` <br>四-数字年份和两个-数字月。| "yyyy-MM" <br>"2019-03" 
`year` <br>`strict_year` <br>四-数字年。| "yyyy" <br>"2019" 
**时代** | 
`time` <br>`strict_time` <br>两个-数字小时，两个-数分钟，两个-数字第二，一个- 到九-数字的一秒钟和时区偏移。| "HH:mm:ss.SSSSSSSSSZ" <br>"21:34:46.123456789-04:00" <br>"21:34:46.1-04:00"
`time_no_millis` <br>`strict_time_no_millis` <br>两个-数字小时，两个-数分钟，两个-数字第二，时区偏移。| "HH:mm:ssZ" <br>"21:34:46-04:00" 
`hour_minute_second_fraction` <br>`strict_hour_minute_second_fraction` <br>两个-数字小时，两个-数分钟，两个-数字第二，一个- 到九-数字的一秒钟。| "HH:mm:ss.SSSSSSSSS" <br>"21:34:46.1" <br>"21:34:46.123456789" 
`hour_minute_second_millis` <br>`strict_hour_minute_second_millis` <br>两个-数字小时，两个-数分钟，两个-数字第二，三个-数字毫秒。| "HH:mm:ss.SSS" <br>"21:34:46.123" 
`hour_minute_second` <br>`strict_hour_minute_second` <br>两个-数字小时，两个-数字分钟和两个-数字第二。| "HH:mm:ss" <br>"21:34:46" 
`hour_minute` <br>`strict_hour_minute` <br>两个-数字小时和两个-数字分钟。| "HH:mm" <br>"21:34" 
`hour` <br>`strict_hour` <br>两个-数字小时。| "HH" <br>"21" 
**T次** |
`t_time` <br>`strict_t_time` <br>两个-数字小时，两个-数分钟，两个-数字第二，一个- 到九-数字的一秒钟和时区偏移，之前`T`。| "`T`HH:mm:ss.SSSSSSSSSZ"<br>"T21:34:46.123456789-04:00" <br>"T21:34:46.1-04:00"
`t_time_no_millis` <br>`strict_t_time_no_millis` <br>两个-数字小时，两个-数分钟，两个-数字第二和时区偏移，之前`T`。| "`T`HH:mm:ssZ" <br>"T21:34:46-04:00"
**序数日期** |
`ordinal_date_time` <br>`strict_ordinal_date_time` <br>由`T`。| "yyyy-DDD`T`HH:mm:ss.SSSZ" <br>"2019-082T21:34:46.123-04:00" 
`ordinal_date_time_no_millis` <br>`strict_ordinal_date_time_no_millis` <br>一个完整的序数日期和时间，没有毫秒`T`。| "yyyy-DDD`T`HH:mm:ssZ" <br>"2019-082T21:34:46-04:00"
`ordinal_date` <br>`strict_ordinal_date`<br>一个四个的完整序数日期-数字年份和三个-一年中的数字序日。| "yyyy-DDD" <br>"2019-082"
**星期-基于日期** |
`week_date_time` <br>`strict_week_date_time` <br>整整一周-基于日期和时间分开`T`。周的日期是四个-数字周-基于一年，两个-一年中的数字序数周，一个-一周中的数字序数。时间是两个-数字小时，两个-数分钟，两个-数字第二，一个- 到九-数字分数为一秒钟，一个时区偏移。| "YYYY-`W`ww-e`T`HH:mm:ss.SSSSSSSSSZ" <br>"2019-W12-6T21:34:46.1-04:00" <br>"2019-W12-6T21:34:46.123456789-04:00"
`week_date_time_no_millis` <br>`strict_week_date_time_no_millis` <br>整整一周-基于没有毫秒的日期和时间`T`。周的日期是四个-数字周-基于一年，两个-一年中的数字序数周，一个-一周中的数字序数。时间是两个-数字小时，两个-数分钟，两个-数字第二，时区偏移。| "YYYY-`W`ww-e`T`HH:mm:ssZ" <br>"2019-W12-6T21:34:46-04:00"
`week_date` <br>`strict_week_date` <br>整整一周-基于四个的日期-数字周-基于一年，两个-一年中的数字序数周，一个-一周中的数字序数。| "YYYY-`W`ww-e" <br>"2019-W12-6"
`weekyear_week_day` <br>`strict_weekyear_week_day` <br>四-数字周-基于一年，两个-一年中的数字序数周，一周中的一个数字。| "YYYY-'W'ww-e" <br>"2019-W12-6" 
`weekyear_week` <br>`strict_weekyear_week` <br>四-数字周-基于年份和两个-一年中的数字序数周。| "YYYY-`W`ww" <br>"2019-W12" 
`weekyear` <br>`strict_weekyear` <br>四-数字周-基于一年。| "YYYY" <br>"2019" 

## 自定义格式

您可以为日期字段创建自定义格式。例如，以下请求指定通用的日期"MM/dd/yyyy" 格式：

```json
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "release_date" : {
        "type" : "date",
        "format" : "MM/dd/yyyy"
      }
    }
  }
}
```
{% include copy-curl.html %}

索引日期的文档：

```json
PUT testindex/_doc/21 
{
  "release_date" : "03/21/2019"
}
```
{% include copy-curl.html %}

搜索确切日期时，以相同格式提供该日期：

```json
GET testindex/_search
{
  "query" : {
    "match": {
      "release_date" : {
        "query": "03/21/2019"
      }
    }
  }
}
```
{% include copy-curl.html %}

默认情况下，范围查询使用该字段的映射格式。您还可以通过提供不同格式的日期范围`format` 范围：

```json
GET testindex/_search
{
  "query": {
    "range": {
      "release_date": {
        "gte": "2019-01-01",
        "lte": "2019-12-31",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 日期数学

日期字段类型使用日期数学支持查询中的持续时间。例如，`gt`，`gte`，`lt`， 和`lte` 参数在[范围查询]({{site.url}}{{site.baseurl}}/query-dsl/term/range/) 和`from` 和`to` 参数在[日期范围聚合]({{site.url}}{{site.baseurl}}/query-dsl/aggregations/bucket/date-range/) 接受日期数学表达式。

日期数学表达式包含固定的日期，然后选择一个或多个数学表达式。固定日期可以是`now` （自时代以来的当前日期和时间为毫秒）或以结尾的字符串`||` 指定日期（例如，`2022-05-18||`）。日期必须在[默认格式](#default-format) （那是`strict_date_time_no_millis||strict_date_optional_time||epoch_millis` 默认情况下）。

如果在字段映射中指定多个日期格式，则OpenSearch使用第一种格式将毫秒转换为字符串，将其转换为毫秒。<br>
如果字段的字段映射不包含格式，则OpenSearch使用`strict_date_optional_time` 格式将时期值转换为字符串。
{: .note}

日期数学支持以下数学运算符。

操作员| 描述| 例子
:--- | :--- | :--- 
`+` | 添加| `+1M`：加1个月。
`-` | 减法| `-1y`：减1年。
`/` | 舍入| `/h`：圆形到一个小时的开始。

日期数学支持以下时间单元：

`y`：年<br>
`M`：月份<br>
`w`：Weeks <br>
`d`：days <br>
`h` 或者`H`：小时<br>
`m`：分钟<br>
`s`：秒
{: .note }

### 示例表达式

以下示例表达式说明了使用日期数学：

- `now+1M`：自时代以来的当前日期和时间为毫秒，再加上1个月。
- `2022-05-18||/M`：05/18/2022，到本月初。决心`2022-05-01`。
- `2022-05-18T15:23||/h`：15：23在05/18/2022，四舍五入到小时的开始。决心`2022-05-18T15`。
- `2022-05-18T15:23:17.789||+2M-1d/d`：15：23：17.789在05/18/2022加上2个月的减去1天，到了一天的开始。决心`2022-07-17`。


### 在范围查询中使用日期数学

以下示例说明了使用日期数学在[范围查询]({{site.url}}{{site.baseurl}}/query-dsl/term/range/)。

与`release_date` 映射为`date`：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "release_date" : {
        "type" : "date"
      }
    }
  }
}
```
{% include copy-curl.html %}

将两个文档索引到索引：

```json
PUT testindex/_doc/1
{
  "release_date": "2022-09-14"
}
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/2
{
  "release_date": "2022-11-15"
}
```
{% include copy-curl.html %}

以下查询搜索文档`release_date` 在09/14/2022的2个月零1天内。该范围的下边界被舍入到09/14/2022的一天开始时：

```json
GET testindex/_search
{
  "query": {
    "range": {
      "release_date": {
        "gte": "2022-09-14T15:23||/d",
        "lte": "2022-09-14||+2M+1d"
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含两个文档：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "release_date" : "2022-11-14"
        }
      },
      {
        "_index" : "testindex",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "release_date" : "2022-09-14"
        }
      }
    ]
  }
}
```

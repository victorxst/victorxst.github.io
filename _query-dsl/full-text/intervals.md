---
layout: default
title: 间隔
nav_order: 80
parent: 全文查询
grand_parent: 查询DSL
---

# 间隔查询

间隔查询根据匹配项的接近性和顺序匹配文档。它将一组_摩擦规则_应用于指定字段中包含的术语。查询生成跨越文本中术语的最小间隔的序列。您可以组合间隔并通过父源过滤它们。

考虑包含以下文档的索引：

```json
PUT testindex/_doc/1 
{
  "title": "key-value pairs are efficiently stored in a hash table"
}
```
{% include copy-curl.html %}

```json
PUT /testindex/_doc/2
{
  "title": "store key-value pairs in a hash map"
}
```
{% include copy-curl.html %}

例如，以下查询搜索包含短语的文档`key-value pairs` （没有差距分开条款），然后是`hash table` 或者`hash map`：

```json
GET /testindex/_search
{
  "query": {
    "intervals": {
      "title": {
        "all_of": {
          "ordered": true,
          "intervals": [
            {
              "match": {
                "query": "key-value pairs",
                "max_gaps": 0,
                "ordered": true
              }
            },
            {
              "any_of": {
                "intervals": [
                  {
                    "match": {
                      "query": "hash table"
                    }
                  },
                  {
                    "match": {
                      "query": "hash map"
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

查询返回两个文档：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 1011,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.25,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.25,
        "_source": {
          "title": "store key-value pairs in a hash map"
        }
      },
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.14285713,
        "_source": {
          "title": "key-value pairs are efficiently stored in a hash table"
        }
      }
    ]
  }
}
```
</details>

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "intervals": {
      "<field>": {
        ... 
      }
    }
  }
}
```
{% include copy-curl.html %}

这`<field>` 接受以下规则对象，这些规则对象根据术语，顺序和接近度匹配文档。

规则| 描述
:--- | :---
[`match`](#the-match-rule) | 匹配分析的文本。
[`prefix`](#the-prefix-rule) | 匹配以指定的字符集开始的术语。
[`wildcard`](#the-wildcard-rule) | 使用通配符模式匹配术语。
[`fuzzy`](#the-fuzzy-rule) | 匹配与指定的编辑距离内提供的项相似的术语。
[`all_of`](#the-all_of-rule) | 使用连词结合多个规则（`AND`）。
[`any_of`](#the-any_of-rule) | 使用析法结合多个规则（`OR`）。

## 这`match` 规则

这`match` 规则匹配分析的文本。下表列出了所有参数`match` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`query` | 必需的| 细绳| 要搜索的文字。
`analyzer` | 选修的| 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于分析`query` 文本。默认是为`<field>`。
[`filter`](#the-filter-rule) | 选修的| 间隔过滤规则对象|  用于过滤返回间隔的规则。
`max_gaps` | 选修的| 整数| 匹配项之间的最大允许数量位置。术语比`max_gaps` 不被视为匹配。如果`max_gaps` 未指定或设置为`-1`，无论其位置如何，术语都被视为匹配。如果`max_gaps` 被设定为`0`，匹配术语必须彼此相邻。默认为`-1`。
`ordered` | 选修的| 布尔| 指定是否必须按照其指定的顺序出现匹配条款。默认为`false`。
`use_field` | 选修的| 细绳| 指定搜索此字段而不是顶部-级<field>。使用针对此字段指定的搜索分析仪对术语进行分析。通过指定`use_field`，您可以跨多个字段搜索，就好像它们都是同一字段一样。例如，如果将相同的文本索引到词干和未茎的字段中，则可以搜索几乎未茎的词干令牌。

## 这`prefix` 规则

这`prefix` 规则匹配以指定的字符集（前缀）开始的术语。前缀可以扩展到最多匹配128个术语。如果前缀匹配超过128个术语，则返回错误。下表列出了所有参数`prefix` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`prefix` | 必需的| 细绳| 前缀用于匹配术语。
`analyzer` | 选修的| 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于标准化`prefix`。默认是为`<field>`。
`use_field` | 选修的| 细绳| 指定搜索此字段而不是顶部-级<field>。这`prefix` 使用针对此字段指定的搜索分析仪进行标准化，除非您指定`analyzer`。

## 这`wildcard` 规则

这`wildcard` 规则使用通配符模式匹配术语。通配符模式可以扩展到最多匹配128个术语。如果模式匹配超过128个术语，则返回错误。下表列出了所有参数`wildcard` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`pattern` | 必需的| 细绳| 通配符模式用于匹配术语。指定`?` 匹配任何单个字符或`*` 匹配零或更多字符。
`analyzer` | 选修的| 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于标准化`pattern`。默认是为`<field>`。
`use_field` | 选修的| 细绳| 指定搜索此字段而不是顶部-级<field>。这`prefix` 使用针对此字段指定的搜索分析仪进行标准化，除非您指定`analyzer`。

指定以开头的模式`*` 或者`?` 可以阻碍搜索性能，因为它增加了匹配项所需的迭代次数。
{: .important}

## 这`fuzzy` 规则

这`fuzzy` 规则匹配与指定的编辑距离内提供的术语相似的术语。模糊模式可以扩展到最多匹配128个术语。如果模式匹配超过128个术语，则返回错误。下表列出了所有参数`fuzzy` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`term` | 必需的| 细绳| 匹配的术语。
`analyzer` | 选修的| 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于标准化`term`。默认是为`<field>`。
`fuzziness` | 选修的| 细绳| 在确定一个术语是否匹配值时，将一个单词更改为另一个单词所需的字符编辑数量（插入，删除，替换）。例如，`wined` 和`wind` 是1.有效值是非-负整数或`AUTO`。默认，`AUTO`，根据每个学期的长度选择一个值，对于大多数用例，是一个不错的选择。
`transpositions` | 选修的| 布尔| 环境`transpositions` 到`true` （默认）在插入，删除和替代操作中添加相邻字符的互换`fuzziness` 选项。例如，`wind` 和`wnid` 是1`transpositions` 是真的（交换"n" 和"i"）和2如果是错误的（删除"n"， 插入"n"）。如果`transpositions` 是`false`，`rewind` 和`wnid` 距离有相同的距离（2）`wind`，尽管人类越来越多-以中心的看法`wnid` 是一个明显的错别字。对于大多数用例，默认值是一个不错的选择。
`prefix_length`| 选修的| 整数| 开始匹配的开始角色数量保持不变。默认值为0。
`use_field` | 选修的| 细绳| 指定搜索此字段而不是顶部-级<field>。这`term` 使用针对此字段指定的搜索分析仪进行标准化，除非您指定`analyzer`。

## 这`all_of` 规则

这`all_of` 规则使用连词结合多个规则（`AND`）。下表列出了所有参数`all_of` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`intervals` | 必需的| 规则对象的数组| 一系列合并的规则。文档必须匹配所有规则，以便在结果中返回。
[`filter`](#the-filter-rule) | 选修的| 间隔过滤规则对象| 用于过滤返回间隔的规则。
`max_gaps` | 选修的| 整数| 匹配项之间的最大允许数量位置。术语比`max_gaps` 不被视为匹配。如果`max_gaps` 未指定或设置为`-1`，无论其位置如何，术语都被视为匹配。如果`max_gaps` 被设定为`0`，匹配术语必须彼此相邻。默认为`-1`。
`ordered` | 选修的| 布尔| 如果`true`，规则生成的间隔应以指定的顺序出现。默认为`false`。

## 这`any_of` 规则

这`any_of` 规则使用脱节结合了多个规则（`OR`）。下表列出了所有参数`any_of` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`intervals` | 必需的| 规则对象的数组| 一系列合并的规则。文档必须至少匹配一个规则才能在结果中返回。
[`filter`](#the-filter-rule) | 选修的| 间隔过滤规则对象| 用于过滤返回间隔的规则。

## 这`filter` 规则

这`filter` 规则用于限制结果。下表列出了所有参数`filter` 规则支持。

范围| 必需/可选| 数据类型| 描述
:--- | :--- | :--- | :---
`after` | 选修的| 查询对象| 用于返回间隔的查询，该间隔遵循过滤器规则中指定的间隔。
`before` | 选修的| 查询对象|  用于返回间隔之前的查询，该间隔是在过滤器规则中指定的。
`contained_by` | 选修的| 查询对象|  用于返回的查询，该查询由过滤器规则中指定的间隔包含。
`containing` | 选修的| 查询对象|  用于返回间隔的查询，该间隔包含过滤器规则中指定的间隔。
`not_contained_by` | 选修的| 查询对象|  用于返回间隔的查询，该间隔未包含过滤器规则中指定的间隔。
`not_containing` | 选修的| 查询对象|  用于返回间隔的查询，该间隔不包含过滤器规则中指定的间隔。
`not_overlapping` | 选修的| 查询对象|  用于返回间隔的查询，该间隔与过滤器规则中指定的间隔不重叠。
`overlapping` | 选修的| 查询对象|  用于返回间隔的查询，该间隔与过滤器规则中指定的间隔重叠。
`script` | 选修的| 脚本对象|  用于匹配文档的脚本。这个脚本必须返回`true` 或者`false`。

#### 示例：过滤器

以下查询搜索包含单词的文档`pairs` 和`hash` 彼此之间的五个位置不包含这个词`efficiently` 它们之间：

```json
POST /testindex/_search
{
  "query": {
    "intervals" : {
      "title" : {
        "match" : {
          "query" : "pairs hash",
          "max_gaps" : 5,
          "filter" : {
            "not_containing" : {
              "match" : {
                "query" : "efficiently"
              }
            }
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

响应仅包含文档2：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.25,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.25,
        "_source": {
          "title": "store key-value pairs in a hash map"
        }
      }
    ]
  }
}
```
</details>

#### 示例：脚本过滤器

另外，您可以编写自己的脚本过滤器`intervals` 使用以下变量查询：

- `interval.start`：间隔启动的位置（项号）。
- `interval.end`：间隔结束的位置（项号）。
- `interval.gap`：术语之间的单词数。

例如，以下查询搜索单词`map` 和`hash` 在指定的间隔内彼此相邻。术语从0开始编号，因此在文本中`store key-value pairs in a hash map`，`store` 在位置0，`key`处于位置`1`， 等等。指定的间隔应在`a` 并在字符串结束之前结束：

```json
POST /testindex/_search
{
  "query": {
    "intervals" : {
      "title" : {
        "match" : {
          "query" : "map hash",
          "filter" : {
            "script" : {
              "source" : "interval.start > 5 && interval.end < 8 && interval.gaps == 0"
            }
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含文件2：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.5,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.5,
        "_source": {
          "title": "store key-value pairs in a hash map"
        }
      }
    ]
  }
}
```
</details>

## 间隔最小化

为了确保查询以线性时间运行，`intervals` 查询最小化的间隔。例如，考虑包含文本的文档`a b c d c`。您可以使用以下查询搜索`d` 由`a` 和`c`：

```json
POST /testindex/_search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "d",
          "filter" : {
            "contained_by" : {
              "match" : {
                "query" : "a c"
              }
            }
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

查询不返回没有结果，因为它与前两个任期匹配`a c` 并发现否`d` 在这些术语之间。


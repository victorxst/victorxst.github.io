---
layout: default
title: 文本分析
has_children: true
nav_order: 5
nav_exclude: true
has_toc: false
redirect_from: 
  - /opensearch/query-dsl/text-analyzers/
  - /query-dsl/analyzers/text-analyzers/
  - /analyzers/text-analyzers/
---

# 文本分析

当您使用完整的文档搜索文档时-文本搜索，您想接收所有相关结果，而不仅仅是确切的匹配。如果您正在寻找"walk"，您对包含任何形式单词的结果感兴趣，"Walk"，"walked"， 或者"walking." 促进饱满-文本搜索，OpenSearch使用文本分析。

文本分析包括以下步骤：

1. _tokenize _ text to术语：例如，象征化后的短语`Actions speak louder than words` 被分成令牌`Actions`，`speak`，`louder`，`than`， 和`words`。
1. _ normalize _术语通过将其转换为标准格式，例如将其转换为小写或执行词干（将单词降低到其根）：例如，在归一化之后，`Actions` 变成`action`，`louder` 变成`loud`， 和`words` 变成`word`。

## 分析仪

在OpenSearch中，文本分析由_analyzer_执行。每个分析仪都包含以下顺序应用的组件：

1. **角色过滤器**：首先，字符过滤器将原始文本作为字符流并添加，删除或修改文本中的字符。例如，字符过滤器可以从字符串中剥离HTML字符，以便文本`<p><b>Actions</b> speak louder than <em>words</em></p>` 变成`\nActions speak louder than words\n`。字符过滤器的输出是字符流。

1. **令牌**：接下来，一个令牌将接收由字符过滤器处理的字符流并将文本拆分为单个_tokens_（通常为单词）。例如，令牌器可以在空白空间上拆分文本，以便前面的文本变为[`Actions`，`speak`，`louder`，`than`，`words`]。Tokenizers还保持有关令牌的元数据，例如其文本中的启动和结束位置。令牌器的输出是代币流。

1. **令牌过滤器**：最后，令牌过滤器从令牌器接收令牌流，并添加，删除或修改令牌。例如，令牌过滤器可能会降低令牌，以便`Actions` 变成`action`，删除像`than`，或添加同义词`talk` 对于这个词`speak`。

分析器必须完全包含一个令牌仪，并且可能包含零或更多字符过滤器和零或更多令牌过滤器。
{: .note}

## 建造-在分析仪中

下表列出了构建的-在OpenSearch提供的分析仪中。表的最后一列包含将分析仪应用于字符串的结果`It’s fun to contribute a brand-new PR or 2 to OpenSearch!`。

分析仪| 进行分析| 分析仪输出
:--- | :--- | :--- 
**标准** （默认）| - 在单词边界处分析字符串<br>- 删除大多数标点符号<br>- 将令牌转换为小写| [`it’s`，`fun`，`to`，`contribute`，`a`，`brand`，`new`，`pr`，`or`，`2`，`to`，`opensearch`这是给出的
**简单的** | - 在任何非-字母字符<br>- 删除非-字母字符<br>- 将令牌转换为小写| [`it`，`s`，`fun`，`to`，`contribute`，`a`，`brand`，`new`，`pr`，`or`，`to`，`opensearch`这是给出的
**空格** | - 在空白空间上解析串入令牌| [`It’s`，`fun`，`to`，`contribute`，`a`，`brand-new`，`PR`，`or`，`2`，`to`，`OpenSearch!`这是给出的
**停止** | - 在任何非-字母字符<br>- 删除非-字母字符<br>- 删除停止单词<br>- 将令牌转换为小写| [`s`，`fun`，`contribute`，`brand`，`new`，`pr`，`opensearch`这是给出的
**关键词** （noop）| - 输出整个字符串不变| [`It’s fun to contribute a brand-new PR or 2 to OpenSearch!`这是给出的
**图案** | - 使用正则表达式分析字符串进入令牌<br>- 支持将字符串转换为小写<br>- 支持删除停止单词| [`it`，`s`，`fun`，`to`，`contribute`，`a`，`brand`，`new`，`pr`，`or`，`2`，`to`，`opensearch`这是给出的
[**语言**]({{site.url}}{{site.baseurl}}/analyzers/language-analyzers/) | 执行特定于某种语言的分析（例如，`english`）。| [`fun`，`contribut`，`brand`，`new`，`pr`，`2`，`opensearch`这是给出的
**指纹** | - 在任何非-字母字符<br>- 通过将字符转换为ASCII <br>来归一化- 将令牌转换为小写<br>- 分类，删除和连续的令牌将令牌纳入一个令牌<br>- 支持删除停止单词| [`2 a brand contribute fun it's new opensearch or pr to`] <br>请注意，撇号转换为ASCII对应物。

## 自定义分析仪

如果需要，您可以将令牌器，令牌过滤器和角色过滤器组合起来，以创建自定义分析器。

## 索引时间和查询时间的文本分析

当您索引文档和发送搜索请求时，OpenSearch在文本字段上执行文本分析。根据文本分析的时间，用于其的分析仪分类如下：

- _index Analyzer_在索引时间进行分析：当您索引时[文本]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 字段，OpenSearch对其进行索引之前对其进行分析。有关指定索引分析仪的方法的更多信息，请参见[索引分析仪]({{site.url}}{{site.baseurl}}/analyzers/index-analyzers/)。

- _search Analyzer_在查询时间进行分析：运行完整时，OpenSearch分析查询字符串-文本字段上的文本查询。有关指定搜索分析仪的方法的更多信息，请参见[搜索分析仪]({{site.url}}{{site.baseurl}}/analyzers/search-analyzers/)。

在大多数情况下，您应该在索引和搜索时间使用相同的分析器，因为文本字段和查询字符串将以相同的方式进行分析，并且所得的令牌将按预期匹配。
{: .tip}

### 例子

当您索引具有文本字段的文档时`Actions speak louder than words`，OpenSearch分析文本并产生以下令牌列表：

文本字段令牌= [`action`，`speak`，`loud`，`than`，`word`这是给出的

当您搜索匹配查询的文档时`speaking loudly`，OpenSearch分析查询字符串并产生以下令牌列表：

查询字符串令牌= [`speak`，`loud`这是给出的

然后OpenSearch将查询字符串中的每个令牌与文本字段令牌列表进行比较，并发现两个列表都包含令牌`speak` 和`loud`，因此OpenSearch将本文档返回，作为与查询匹配的搜索结果的一部分。

## 测试分析仪

测试建造的-在分析仪中并查看文档索引时生成的令牌列表，您可以使用[分析API]({{site.url}}{{site.baseurl}}/api-reference/analyze-apis/#apply-a-built-in-analyzer)。

指定分析仪和要在请求中分析的文本：

```json
GET /_analyze
{
  "analyzer" : "standard",
  "text" : "Let’s contribute to OpenSearch!"
}
```
{% include copy-curl.html %}

下图显示了查询字符串。

![查询字符串带索引]({{site.url}}{{site.baseurl}}/images/string-indices.png)

该响应包含每个令牌及其开始和端的偏移，这些响应与原始字符串（包含）和结束索引（独家）中的起始索引相对应：

```json
{
  "tokens": [
    {
      "token": "let’s",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "contribute",
      "start_offset": 6,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "to",
      "start_offset": 17,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "opensearch",
      "start_offset": 20,
      "end_offset": 30,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```

## 验证分析仪设置

为了验证哪个分析仪与哪个字段相关联，您可以使用get映射API操作：

```json
GET /testindex/_mapping
```
{% include copy-curl.html %}

该响应提供了有关每个字段分析仪的信息：

```json
{
  "testindex": {
    "mappings": {
      "properties": {
        "text_entry": {
          "type": "text",
          "analyzer": "simple",
          "search_analyzer": "whitespace"
        }
      }
    }
  }
}
```

## 下一步

- 了解有关指定的更多信息[索引分析仪]({{site.url}}{{site.baseurl}}/analyzers/index-analyzers/) 和[搜索分析仪]({{site.url}}{{site.baseurl}}/analyzers/search-analyzers/)


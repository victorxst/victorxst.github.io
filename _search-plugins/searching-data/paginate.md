---
layout: default
title: 分页效果
parent: 搜索数据
nav_order: 21
redirect_from:
  - /opensearch/search/paginate/
---

## 分页效果

您可以使用以下方法在OpenSearch中分页搜索结果：

1. 这[`from` 和`size` 参数](#the-from-and-size-parameters)
1. 这[滚动搜索](#scroll-search) 手术
1. 这[`search_after` 范围](#the-search_after-parameter)
1. [时间点`search_after`](#point-in-time-with-search_after)

## 这`from` 和`size` 参数

这`from` 和`size` 参数一次返回结果一页。

这`from` 参数是您要从中开始显示结果的文档编号。这`size` 参数是您要显示的结果数。他们一起让您返回搜索结果的子集。

例如，如果价值`size` 是10，价值`from` 是0，您会看到前10个结果。如果您更改价值`from` 到10，您会看到接下来的10个结果（因为结果为零-索引）。因此，如果您想从结果11开始查看结果，`from` 必须是10。

```json
GET shakespeare/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match": {
      "play_name": "Hamlet"
    }
  }
}
```

使用以下公式计算`from` 相对于页码的参数：

```json
from = size * (page_number - 1)
```

每次用户选择结果的下一页时，您的应用程序都需要以增量运行相同的搜索查询`from` 价值。

您也可以指定`from` 和`size` 搜索URI中的参数：

```json
GET shakespeare/_search?from=0&size=10
```

如果您仅指定`size` 参数，`from` 参数默认为0。

在您的结果深处查询页面可能会产生重大的性能影响，因此OpenSearch将这种方法限制为10,000个结果。

这`from` 和`size` 参数是无状态的，因此结果基于最新的可用数据。
这可能会导致分页不一致。
例如，假设用户停留在结果的第一页上，然后导航到第二页。在此期间，索引了与用户搜索相关的新文档，并显示在第一页上。在这种情况下，将第一页的最后一个结果推到第二页，用户看到重复的结果（即第一页和第二页都显示了最后一个结果）。

使用`scroll` 操作用于一致的分页。这`scroll` 操作可以使搜索环境在一定时间段内打开。在此期间，任何数据更改都不会影响结果。


## 滚动搜索

这`from` 和`size` 参数允许您分页搜索结果，但一次限制10,000个结果。

如果您需要要求大于1 pb的数据量，例如，机器学习工作，请使用`scroll` 而是操作。这`scroll` 操作使您可以要求无限数量的结果。

要使用滚动操作，请添加一个`scroll` 通过搜索上下文告诉OpenSearch的参数，请搜索您需要滚动多长时间。此搜索上下文需要足够长的时间来处理一批结果。

要设置您要返回每批结果的结果数，请使用`size` 范围：

```json
GET shakespeare/_search?scroll=10m
{
  "size": 10000
}
```

OpenSearch缓存结果并返回滚动ID，您可以用来批量访问它们：

```json
"_scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAUWdmpUZDhnRFBUcWFtV21nMmFwUGJEQQ=="
```

将此卷轴ID传递给`scroll` 以获取下一批结果的操作：

```json
GET _search/scroll
{
  "scroll": "10m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAUWdmpUZDhnRFBUcWFtV21nMmFwUGJEQQ=="
}
```

使用此滚动ID，只要搜索上下文仍然打开，您就可以将结果批量为10,000。通常，滚动ID不会在请求之间发生变化，但是 *可以 *更改，因此请确保始终使用最新的滚动ID。如果您不在设定搜索上下文中发送下一个滚动请求，则`scroll` 操作不会返回任何结果。

如果您期望数十亿个结果，请使用切片卷轴。切片使您可以针对相同请求执行多个滚动操作，但并行。
设置ID和滚动的最大切片数：

```json
GET shakespeare/_search?scroll=10m
{
  "slice": {
    "id": 0,
    "max": 10
  },
  "query": {
    "match_all": {}
  }
}
```

使用单个滚动ID，您将获得10个结果。
您最多可以拥有10个ID。
用等于1的ID执行相同的命令：

```json
GET shakespeare/_search?scroll=10m
{
  "slice": {
    "id": 1,
    "max": 10
  },
  "query": {
    "match_all": {}
  }
}
```

完成滚动后关闭搜索上下文，因为它继续消耗计算资源直到超时为止：

```json
DELETE _search/scroll/DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAcWdmpUZDhnRFBUcWFtV21nMmFwUGJEQQ==
```

#### 样本响应

```json
{
  "succeeded": true,
  "num_freed": 1
}
```

使用以下请求来关闭所有打开的滚动上下文：

```json
DELETE _search/scroll/_all
```

这`scroll` 操作对应于特定的时间戳。它不会将时间戳之后添加的文档视为潜在结果。

因为开放搜索上下文会消耗大量内存，我们建议您不要使用`scroll` 操作用于频繁的用户查询，而用户查询不需要打开搜索上下文。而是使用`sort` 带有的参数`search_after` 参数以滚动响应用户查询。

## 这`search_after` 范围

这`search_after` 参数提供了一个实时光标，该光标使用上一页的结果获得下一页的结果。它类似于`scroll` 操作是要并行滚动许多查询。

例如，以下查询播放的所有行"Hamlet" 根据语音号码，然后通过ID检索前三个结果：

```json
GET shakespeare/_search
{
  "size": 3,
  "query": {
    "match": {
      "play_name": "Hamlet"
    }
  },
  "sort": [
    { "speech_number": "asc" },
    { "_id": "asc" } 
  ]
}
```

响应包含`sort` 每个文档的值数组：

```json
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4244,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "32435",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 32436,
          "play_name" : "Hamlet",
          "speech_number" : 1,
          "line_number" : "1.1.1",
          "speaker" : "BERNARDO",
          "text_entry" : "Whos there?"
        },
        "sort" : [
          1,
          "32435"
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "32634",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 32635,
          "play_name" : "Hamlet",
          "speech_number" : 1,
          "line_number" : "1.2.1",
          "speaker" : "KING CLAUDIUS",
          "text_entry" : "Though yet of Hamlet our dear brothers death"
        },
        "sort" : [
          1,
          "32634"
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "32635",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 32636,
          "play_name" : "Hamlet",
          "speech_number" : 1,
          "line_number" : "1.2.2",
          "speaker" : "KING CLAUDIUS",
          "text_entry" : "The memory be green, and that it us befitted"
        },
        "sort" : [
          1,
          "32635"
        ]
      }
    ]
  }
}
```

您可以使用最后结果`sort` 通过使用`search_after` 范围：

```json
GET shakespeare/_search
{
  "size": 10,
  "query": {
    "match": {
      "play_name": "Hamlet"
    }
  },
  "search_after": [ 1, "32635"],
  "sort": [
    { "speech_number": "asc" },
    { "_id": "asc" } 
  ]
}
```

不像`scroll` 操作，`search_after` 参数无状态，因此由于索引或删除的文档，文档订单可能会更改。

## 时间点`search_after`

时间点（坑）`search_after` 是Opensearch中的首选分页方法，尤其是对于深层分页。它绕过了所有其他方法的局限性，因为它在及时冻结的数据集上运行，它不受查询的限制，并且支持一致的分页，向前又向后。要了解更多，请参阅[时间点]({{site.url}}{{site.baseurl}}/opensearch/point-in-time)


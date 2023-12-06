---
layout: default
title: 滚动
nav_order: 71
redirect_from:
 - /opensearch/rest-api/scroll/
---

# 滚动
**引入1.0**
{: .label .label-purple }

您可以使用`scroll` 操作以检索大量结果。例如，对于机器学习工作，您可以分批索取无限数量的结果。

使用`scroll` 操作，添加一个`scroll` 使用搜索上下文的请求标头的参数，以告诉OpenSearch您需要滚动多长时间。此搜索上下文需要足够长的时间来处理一批结果。

因为搜索上下文会消耗大量内存，我们建议您不要使用`scroll` 用于频繁的用户查询的操作。而是使用`sort` 带有的参数`search_after` 参数以滚动响应用户查询。
{:.note}

## 例子

要设置您要返回每批结果的结果数，请使用`size` 范围：

```json
GET shakespeare/_search?scroll=10m
{
  "size": 10000
}
```
{% include copy-curl.html %}

OpenSearch缓存结果并返回滚动ID，以分批访问它们：

```json
"_scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAUWdmpUZDhnRFBUcWFtV21nMmFwUGJEQQ=="
```

将此卷轴ID传递给`scroll` 恢复下一批结果的操作：

```json
GET _search/scroll
{
  "scroll": "10m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAUWdmpUZDhnRFBUcWFtV21nMmFwUGJEQQ=="
}
```
{% include copy-curl.html %}

使用此滚动ID，只要搜索上下文仍然打开，您就可以将结果批量为10,000。通常，滚动ID不会在请求之间发生变化，但是 *可以 *更改，因此请确保始终使用最新的滚动ID。如果您不在设定搜索上下文中发送下一个滚动请求，则`scroll` 操作不会返回任何结果。

如果您期望数十亿个结果，请使用切片卷轴。切片允许您针对相同请求执行多个滚动操作，但并行。
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
{% include copy-curl.html %}

有了一个滚动ID，您将获得10个结果。
您最多可以拥有10个ID。

完成滚动后关闭搜索上下文，因为`scroll` 操作继续消耗计算资源，直到超时为止：

```json
DELETE _search/scroll/DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAcWdmpUZDhnRFBUcWFtV21nMmFwUGJEQQ==
```
{% include copy-curl.html %}

关闭所有打开卷轴上下文：

```json
DELETE _search/scroll/_all
```
{% include copy-curl.html %}

这`scroll` 操作对应于特定的时间戳。它不会将时间戳之后添加的文档视为潜在结果。


## 路径和HTTP方法

```
GET _search/scroll
POST _search/scroll
```
```
GET _search/scroll/<scroll-id>
POST _search/scroll/<scroll-id>
```

## URL参数

所有滚动参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
滚动| 时间| 指定维持搜索上下文的时间。
scroll_id| 细绳| 搜索的滚动ID。
REST_TOTAL_HITS_AS_INT| 布尔| 是否`hits.total` 属性作为整数返回（`true`）或一个对象（`false`）。默认为`false`。

## 回复

```json
{
  "succeeded": true,
  "num_freed": 1
}
```


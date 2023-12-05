---
layout: default
title: 时间点
nav_order: 58
has_children: true
has_toc: false
redirect_from:
  - /opensearch/point-in-time/
---

# 时间点

时间点（PIT）使您可以针对固定时间固定的数据集运行不同的查询。

通常，如果您多次在索引上运行查询，则相同的查询可能会返回不同的结果，因为文档是不断索引，更新和删除的。如果您需要针对相同数据运行查询，则可以通过创建一个坑来保留该数据的状态。坑功能的主要用途是将其与`search_after` 深度分页的搜索结果的功能。

## 登上搜索结果

除了坑功能外，还有三种方法可以[分页搜索结果]({{site.url}}{{site.baseurl}}/opensearch/search/paginate) 在OpenSearch中：使用滚动API，指定`from` 和`size` 搜索的参数，并使用`search_after` 功能。但是，这三个都有局限性：

- 滚动API的搜索结果在请求时被冻结，但它们绑定到特定的查询。此外，滚动只能在搜索中向前移动，因此，如果页面请求失败，则随后的请求跳过该页面并返回以下内容。
- 如果指定`from` 和`size` 搜索的参数，搜索结果未及时冻结，因此由于索引或删除了文档，它们可能不一致。这`from` 和`size` 不建议使用功能进行深度分页，因为每个页面请求都需要处理所有结果并将其过滤为请求的页面。
- 这`search_after` 搜索结果没有及时冷冻，因此由于同时进行的文档索引或删除，它们可能不一致。

PIT功能没有其他分页方法的局限性，因为PIT搜索不受查询的绑定，并且支持一致的分页，向前又向后进行。如果您查看了您的结果的第一页，并且现在在第二页上，则如果您返回，您将看到同一页第一页。

## 坑搜索

PIT搜索具有与常规搜索相同的功能，除了PIT搜索在较旧的数据集上行动，而定期搜索在实时数据集上进行。PIT搜索不绑定到查询，因此您可以在及时冻结的同一数据集上运行不同的查询。

您可以使用[创建PIT API]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api#create-a-pit) 创建一个坑。当您为一组索引创建一个坑时，请搜索锁定这些索引的一组段，并及时冻结它们。在较低的级别上，该坑所需的资源均未修改或删除。如果合并了坑的一部分，OpenSearch保留了这些细分市场的副本，则在Pit创建时指定的时间段`keep_alive` 范围。

Create Pit Operation返回一个PIT ID，您可以在冷冻数据集上运行多个查询。即使该索引继续摄入数据并修改或删除文档，坑也引用了自维修区创建以来未更改的数据。当查询包含一个坑ID时，您无需将索引传递给搜索，因为它将使用该坑。当您多次运行时，具有PIT ID的搜索将产生完全相同的结果。

如果发生群集或节点故障，则所有PIT数据都会丢失。
{: .note}

## 与坑和搜索的分页

当您运行带有坑ID的查询时，您可以使用`search_after` 参数以检索结果的下一页。这使您可以控制结果页面中的文档顺序。

使用PIT ID运行搜索查询：

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
    "keep_alive": "100m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc"}}
  ]
}
```

响应包含与查询相匹配的前10,000个文档。要获取下一组文档，请以最后一个文档的排序值与`search_after` 参数，保持相同`sort` 和`pit.id`。您可以使用可选的`keep_alive` 延长进站时间的参数：

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
    "keep_alive": "100m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc"}}
  ],
  "search_after": [  
    "2021-05-20T05:30:04.832Z"
  ]
}
```

## 搜索切片

使用`search_after` 使用凹坑进行分页，您可以控制结果的订购。如果您不需要任何特定顺序的结果，或者您希望能够从页面跳到非页面-连续页面，您可以使用搜索切片。搜索切片将坑搜索分为多个切片，这些切片可以通过客户端应用程序独立消费。

例如，如果您有一个有1,000,000个结果的坑搜索查询，并且您想一次返回50,000个结果，则您的客户端应用程序必须连续20个电话以接收每批结果。如果使用搜索切片，则可以并行化这20个呼叫。在您的多线程客户端应用程序中，您可以为每个坑使用五个切片。结果，您将拥有5 10,000-点击切片可以被客户端中的五个不同线程消耗，而不是让单个线程消耗50,000个结果。

要使用搜索切片，您必须指定两个参数：
- `slice.id` 是您要求的切片ID。
- `slice.max` 是将搜索响应分解成的切片数。

以下坑搜索查询说明了搜索切片：

```json

GET /_search
{
  "slice": {
    "id": 0,  // id is the slice (page) number being requested. In every request we can only query for one slice                    
    "max": 2  // max is the total number of slices (pages) the search response will be broken down into                  
  },
  "query": {
    "match": {
      "message": "foo"
    }
  },
  "pit": {
    "id": "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
  }
}
```

在每个请求中，您只能查询一个切片，因此下一个查询将与上一个查询相同，除了`slice.id` 将`1`。

## 安全模型

本节介绍如果您正在使用启用安全插件运行OpenSearch，则使用PIT API操作所需的权限。

用户可以使用`point_in_time_full_access` 角色。如果此角色无法满足您的需求，请混合并匹配单个坑的权限以适合您的用例。每个动作对应于REST API中的操作。例如，`indices:data/read/point_in_time/create` 许可使您可以创建一个坑。以下是可能的权限：

- `indices:data/read/point_in_time/create` ＆ndash;创建API
- `indices:data/read/point_in_time/delete` ＆ndash;删除API
- `indices:data/read/point_in_time/readall` ＆ndash;列出所有凹坑API
- `indices:data/read/search` ＆ndash;搜索API
- `indices:monitor/point_in_time/segments` ＆ndash;坑段API

为了`all` API操作，例如list All和删除ALL，用户需要所有索引（*）权限。对于API操作，例如搜索，创建PIT或删除列表，用户只需要单个索引权限。

保存时，坑ID始终包含基础（已解决）索引。以下各节描述了别名和数据流的所需权限。

### 别名许可

对于别名，用户必须具有索引**或者** 任何坑操作的别名许可。

### 数据流权限

对于数据流，用户必须两个数据流**和** 数据流的任何维修操作的备份索引权限。例如，用户必须具有`data-stream-11` 数据流及其支持索引`.ds-my-data-stream11-000001`。

如果用户仅具有数据流许可，则他们将能够创建一个坑，但是他们将无法将PIT ID用于其他操作，例如搜索，而无需备份索引权限。

## API

下表列出了所有[时间点API]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api) 功能。

功能| API| 描述
:--- | :--- | :---
[创建坑]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api#create-a-pit) | `POST /<target_indexes>/_search/point_in_time?keep_alive=1h` | 创建一个坑。
[列表坑]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api#list-all-pits) | `GET /_search/point_in_time/_all` | 列出所有坑。
[删除坑]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api#delete-pits) | `DELETE /_search/point_in_time`<br>`DELETE /_search/point_in_time/_all` | 删除一个坑或所有坑。
[坑段]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api#pit-segments) | `GET /_cat/pit_segments/_all` | 通过描述其Lucene段，提供有关磁盘利用磁盘利用的信息。

有关相关群集和节点设置的信息，请参见[维修区设置]({{site.url}}{{site.baseurl}}/opensearch/point-in-time-api#pit-settings)。


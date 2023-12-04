---
layout: default
title: Multi-search
nav_order: 45
redirect_from: 
 - /opensearch/rest-api/multi-search/
---

# 并发-搜索
**引入1.0**
{: .label .label-purple }

顾名思义，并发-搜索操作使您可以将多个搜索请求捆绑到一个请求中。然后，OpenSearch并并行执行搜索，因此与每次搜索发送一个请求相比，您更快地获得了响应。OpenSearch独立执行每个搜索，因此一个失败不会影响其他搜索。

## 例子

```json
GET _msearch
{ "index": "opensearch_dashboards_sample_data_logs"}
{ "query": { "match_all": {} }, "from": 0, "size": 10}
{ "index": "opensearch_dashboards_sample_data_ecommerce", "search_type": "dfs_query_then_fetch"}
{ "query": { "match_all": {} } }

```
{% include copy-curl.html %}


## 路径和HTTP方法

```
GET _msearch
GET <indices>/_msearch
POST _msearch
POST <indices>/_msearch
```


## 请求身体

并发-搜索请求主体遵循以下模式：

```
Metadata\n
Query\n
Metadata\n
Query\n

```

- 元数据线包括选项，例如搜索索引和搜索类型。
- 查询线使用[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/)。

就像[大部分]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/) 操作，JSON不需要缩小---空间很好---但是它确实需要在一条线上。OpenSearch使用newline字符来解析元-搜索请求，并要求请求主体以newline字符结束。


## URL参数和元数据选项

全部并发-搜索URL参数是可选的。有些也可以使用-作为每条元数据线的一部分搜索。

范围| 类型| 描述| 元数据线支持
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认为`true`。| 是的
cancel_after_time_interval| 时间| 之后将取消搜索请求的时间。在父母和子女请求级别上得到支持。优先顺序是：<br> 1.孩子-级别参数<br> 2.父-级别参数<br> 3。[集群设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-settings)。<br>默认值为-1。| 是的
CSS_MINIMIZE_ROUNDTRIPS| 布尔| OpenSearch是否应该尝试最大程度地减少协调节点和远程群集之间的网络往返数量（仅适用于交叉-集群搜索请求）。默认为`true`。| 不
Expand_WildCard| 枚举| 将通配符表达式扩展到混凝土指数。将多个值与逗号相结合。支持的值是`all`，`open`，`closed`，`hidden`， 和`none`。默认为`open`。| 是的
ignore_unavailable| 布尔| 如果不存在来自索引列表的索引，是否忽略它而不是使查询失败。默认为`false`。| 是的
max_concurrent_searches| 整数| 并发搜索的最大数量。默认值取决于您的节点计数和搜索线程池大小。较高的值可以提高性能，但有可能使集群超负荷。| 不
max_concurrent_shard_requests| 整数| 每个节点执行每个搜索执行的并发碎片请求的最大数量。默认值为5。更高的值可以提高性能，但风险超载集群。| 不
pre_filter_shard_size| 整数| 默认值为128。| 不
REST_TOTAL_HITS_AS_INT| 细绳| 是否`hits.total` 属性作为整数返回（`true`）或一个对象（`false`）。默认为`false`。| 不
搜索类型| 细绳| 影响相关得分。有效的选项是`query_then_fetch` 和`dfs_query_then_fetch`。`query_then_fetch` 使用术语和文档频率分数（更快，准确）的频率分数，而`dfs_query_then_fetch` 在所有碎片中使用术语和文档频率（较慢，更准确）。默认为`query_then_fetch`。| 是的
typed_keys| 布尔| 是否在响应中将聚合名称添加到其内部类型。默认为`false`。| 不

{％评论％}`pre_filter_shard_size`：来自REST API规范的描述难以理解---无论如何，对我来说。我也无法从阅读源代码中学到任何东西，因此我在上表中包含了默认值，没有其他内容。- Aetter

从REST API规范中：执行PRE的阈值-如果搜索请求扩展到超过阈值，则根据查询重写的查询重写往返前过滤器搜索碎片。如果碎片无法根据其重写方法IE匹配任何文档，则此过滤器往返可以显着限制碎片数。如果必须匹配日期过滤器，但是碎片范围和查询是不相交的。


## 元数据-只有选项

某些选项不能作为URL参数应用于整个请求。相反，您可以使用它们-作为每条元数据线的一部分搜索。所有都是可选的。

选项| 类型| 描述
:--- | :--- | :---
指数| 字符串，字符串数组| 如果您不作为URL的一部分指定索引或多个索引（或想覆盖单个搜索的URL值），则可以在此处包含它。示例包括`"logs-*"` 和`["my-store", "sample_data_ecommerce"]`。
偏爱| 细绳| 您想执行搜索的节点或碎片。此设置对于测试可能很有用，但是在大多数情况下，默认行为提供了最佳的搜索潜伏期。选项包括`_local`，`_only_local`，`_prefer_nodes`，`_only_nodes`， 和`_shards`。这些最后三个选项接受节点或碎片列表。示例包括`"_only_nodes:data-node1,data-node2"` 和`"_shards:0,1`。
request_cache| 布尔| 是否要缓存结果，可以改善重复搜索的延迟。默认是使用`index.requests.cache.enable` 设置索引（默认为`true` 对于新索引）。
路由| 细绳| 逗号-分开的自定义路由值，例如`"routing": "value1,value2,value3"`。


## 回复

OpenSearch返回一个数组，每个搜索的结果都以与Multi相同的顺序返回-搜索请求。

```json
{
  "took" : 2150,
  "responses" : [
    {
      "took" : 2149,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 10000,
          "relation" : "gte"
        },
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : "opensearch_dashboards_sample_data_logs",
            "_id" : "_fnhBXsBgv2Zxgu9dZ8Y",
            "_score" : 1.0,
            "_source" : {
              "agent" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
              "bytes" : 4657,
              "clientip" : "213.116.129.196",
              "extension" : "zip",
              "geo" : {
                "srcdest" : "CN:US",
                "src" : "CN",
                "dest" : "US",
                "coordinates" : {
                  "lat" : 42.35083333,
                  "lon" : -86.25613889
                }
              },
              "host" : "artifacts.opensearch.org",
              "index" : "opensearch_dashboards_sample_data_logs",
              "ip" : "213.116.129.196",
              "machine" : {
                "ram" : 16106127360,
                "os" : "ios"
              },
              "memory" : null,
              "message" : "213.116.129.196 - - [2018-07-30T14:12:11.387Z] \"GET /opensearch_dashboards/opensearch_dashboards-1.0.0-windows-x86_64.zip HTTP/1.1\" 200 4657 \"-\" \"Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)\"",
              "phpmemory" : null,
              "referer" : "http://twitter.com/success/ellison-onizuka",
              "request" : "/opensearch_dashboards/opensearch_dashboards-1.0.0-windows-x86_64.zip",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-08-02T14:12:11.387Z",
              "url" : "https://artifacts.opensearch.org/downloads/opensearch_dashboards/opensearch_dashboards-1.0.0-windows-x86_64.zip",
              "utc_time" : "2021-08-02T14:12:11.387Z",
              "event" : {
                "dataset" : "sample_web_logs"
              }
            }
          },
          ...
        ]
      },
      "status" : 200
    },
    {
      "took" : 1473,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 4675,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : "opensearch_dashboards_sample_data_ecommerce",
            "_id" : "efnhBXsBgv2Zxgu9ap7e",
            "_score" : 1.0,
            "_source" : {
              "category" : [
                "Women's Clothing"
              ],
              "currency" : "EUR",
              "customer_first_name" : "Gwen",
              "customer_full_name" : "Gwen Dennis",
              "customer_gender" : "FEMALE",
              "customer_id" : 26,
              "customer_last_name" : "Dennis",
              "customer_phone" : "",
              "day_of_week" : "Tuesday",
              "day_of_week_i" : 1,
              "email" : "gwen@dennis-family.zzz",
              "manufacturer" : [
                "Tigress Enterprises",
                "Gnomehouse mom"
              ],
              "order_date" : "2021-08-10T16:24:58+00:00",
              "order_id" : 576942,
              "products" : [
                {
                  "base_price" : 32.99,
                  "discount_percentage" : 0,
                  "quantity" : 1,
                  "manufacturer" : "Tigress Enterprises",
                  "tax_amount" : 0,
                  "product_id" : 22182,
                  "category" : "Women's Clothing",
                  "sku" : "ZO0036600366",
                  "taxless_price" : 32.99,
                  "unit_discount_amount" : 0,
                  "min_price" : 14.85,
                  "_id" : "sold_product_576942_22182",
                  "discount_amount" : 0,
                  "created_on" : "2016-12-20T16:24:58+00:00",
                  "product_name" : "Jersey dress - black/red",
                  "price" : 32.99,
                  "taxful_price" : 32.99,
                  "base_unit_price" : 32.99
                },
                {
                  "base_price" : 28.99,
                  "discount_percentage" : 0,
                  "quantity" : 1,
                  "manufacturer" : "Gnomehouse mom",
                  "tax_amount" : 0,
                  "product_id" : 14230,
                  "category" : "Women's Clothing",
                  "sku" : "ZO0234902349",
                  "taxless_price" : 28.99,
                  "unit_discount_amount" : 0,
                  "min_price" : 13.05,
                  "_id" : "sold_product_576942_14230",
                  "discount_amount" : 0,
                  "created_on" : "2016-12-20T16:24:58+00:00",
                  "product_name" : "Blouse - june bug",
                  "price" : 28.99,
                  "taxful_price" : 28.99,
                  "base_unit_price" : 28.99
                }
              ],
              "sku" : [
                "ZO0036600366",
                "ZO0234902349"
              ],
              "taxful_total_price" : 61.98,
              "taxless_total_price" : 61.98,
              "total_quantity" : 2,
              "total_unique_products" : 2,
              "type" : "order",
              "user" : "gwen",
              "geoip" : {
                "country_iso_code" : "US",
                "location" : {
                  "lon" : -118.2,
                  "lat" : 34.1
                },
                "region_name" : "California",
                "continent_name" : "North America",
                "city_name" : "Los Angeles"
              },
              "event" : {
                "dataset" : "sample_ecommerce"
              }
            }
          },
         ...
        ]
      },
      "status" : 200
    }
  ]
}
```


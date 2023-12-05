---
layout: default
title: Ruby客户端 
nav_order: 60
has_children: false
---

# Ruby客户端

OpenSearch Ruby客户端使您可以通过Ruby方法而不是HTTP方法和RAW JSON与OpenSearch群集进行交互。有关客户的完整API文档和其他示例，请参见[`opensearch-transport`](https://rubydoc.info/gems/opensearch-transport)，[`opensearch-api`](https://rubydoc.info/gems/opensearch-api)，[`opensearch-dsl`](https://rubydoc.info/gems/opensearch-dsl)， 和[`opensearch-ruby`](https://rubydoc.info/gems/opensearch-ruby/) 宝石文档。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-Ruby Repo](https://github.com/opensearch-project/opensearch-ruby)。

## 安装Ruby客户端

要为Ruby客户端安装Ruby Gem，请运行以下命令：

```bash
gem install opensearch-ruby
```
{% include copy.html %}

要使用客户端，请将其导入一个模块：

```ruby
require 'opensearch'
```
{% include copy.html %}

## 连接到OpenSearch

要连接到默认的OpenSearch主机，请创建一个客户端对象，传递构造函数中的默认主机地址：

```ruby
client = OpenSearch::Client.new(host: 'http://localhost:9200')
```
{% include copy.html %}

以下示例使用自定义URL创建客户端对象和`log` 选项设置为`true`。它设置了`retry_on_failure` 重试失败请求的参数五次而不是默认的三次。最后，通过设置`request_timeout` 参数至120秒。然后，它返回基本的群集健康信息：

```ruby
client = OpenSearch::Client.new(
    url: "http://localhost:9200",
    retry_on_failure: 5,
    request_timeout: 120,
    log: true
  )

client.cluster.health
```
{% include copy.html %}

输出如下：

```bash
2022-08-25 14:24:52 -0400: GET http://localhost:9200/ [status:200, request:0.048s, query:n/a]
2022-08-25 14:24:52 -0400: < {
  "name" : "opensearch",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "Aw0F5Pt9QF6XO9vXQHIs_w",
  "version" : {
    "distribution" : "opensearch",
    "number" : "2.2.0",
    "build_type" : "tar",
    "build_hash" : "b1017fa3b9a1c781d4f34ecee411e0cdf930a515",
    "build_date" : "2022-08-09T02:27:25.256769336Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.10.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}

2022-08-25 14:24:52 -0400: GET http://localhost:9200/_cluster/health [status:200, request:0.018s, query:n/a]
2022-08-25 14:24:52 -0400: < {"cluster_name":"docker-cluster","status":"yellow","timed_out":false,"number_of_nodes":1,"number_of_data_nodes":1,"discovered_master":true,"discovered_cluster_manager":true,"active_primary_shards":10,"active_shards":10,"relocating_shards":0,"initializing_shards":0,"unassigned_shards":8,"delayed_unassigned_shards":0,"number_of_pending_tasks":0,"number_of_in_flight_fetch":0,"task_max_waiting_in_queue_millis":0,"active_shards_percent_as_number":55.55555555555556}
```

## 连接到Amazon OpenSearch服务

要连接到Amazon OpenSearch服务，请首先安装`opensearch-aws-sigv4` 宝石：

```bash
gem install opensearch-aws-sigv4
```

```ruby
require 'opensearch-aws-sigv4'
require 'aws-sigv4'

signer = Aws::Sigv4::Signer.new(service: 'es',
                                region: 'us-west-2', # signing service region
                                access_key_id: 'key_id',
                                secret_access_key: 'secret')

client = OpenSearch::Aws::Sigv4Client.new({
    host: 'https://your.amz-managed-opensearch.domain',
    log: true
}, signer)

# create an index and document
index = 'prime'
client.indices.create(index: index)
client.index(index: index, id: '1', body: { name: 'Amazon Echo', 
                                            msrp: '5999', 
                                            year: 2011 })

# search for the document
client.search(body: { query: { match: { name: 'Echo' } } })

# delete the document
client.delete(index: index, id: '1')

# delete the index
client.indices.delete(index: index)
```
{% include copy.html %}

## 连接到Amazon OpenSearch无服务器

要连接到Amazon OpenSearch无用服务，请首先安装`opensearch-aws-sigv4` 宝石：

```bash
gem install opensearch-aws-sigv4
```

```ruby
require 'opensearch-aws-sigv4'
require 'aws-sigv4'

signer = Aws::Sigv4::Signer.new(service: 'aoss',
                                region: 'us-west-2', # signing service region
                                access_key_id: 'key_id',
                                secret_access_key: 'secret')

client = OpenSearch::Aws::Sigv4Client.new({
    host: 'https://your.amz-managed-opensearch.domain', # serverless endpoint for OpenSearch Serverless
    log: true
}, signer)

# create an index and document
index = 'prime'
client.indices.create(index: index)
client.index(index: index, id: '1', body: { name: 'Amazon Echo', 
                                            msrp: '5999', 
                                            year: 2011 })

# search for the document
client.search(body: { query: { match: { name: 'Echo' } } })

# delete the document
client.delete(index: index, id: '1')

# delete the index
client.indices.delete(index: index)
```
{% include copy.html %}


## 创建索引

您无需在OpenSearch中明确创建索引。一旦将文档上传到不存在的索引中，OpenSearch就会自动创建索引。另外，您可以明确创建一个索引，以指定设置，例如主和复制碎片的数量。用非-默认设置，使用这些设置创建索引主体哈希：

```ruby
index_body = {
    'settings': {
        'index': {
        'number_of_shards': 1,
        'number_of_replicas': 2 
        }
    }
} 

client.indices.create(
    index: 'students',
    body: index_body
)
```
{% include copy.html %}

## 映射

OpenSearch使用动态映射来推断索引的文档的字段类型。但是，要对文档的架构进行更多控制，您可以将明确的映射传递给OpenSearch。您可以在此映射中为文档的某些字段定义数据类型。要为索引创建映射，请使用`put_mapping` 方法：

```ruby
client.indices.put_mapping(
    index: 'students', 
    body: {
        properties: {
            first_name: { type: 'keyword' },
            last_name: { type: 'keyword' }
        }  
    }
)
```
{% include copy.html %}

默认情况下，字符串字段映射为`text`，但在上面的映射中`first_name` 和`last_name` 字段被映射为`keyword`。该映射信号向OpenSearch进行了搜索，说这些字段不应进行分析，并且仅支持完整案例-敏感匹配。

您可以使用该索引来验证索引的映射`get_mapping` 方法：

```ruby
response = client.indices.get_mapping(index: 'students')
```
{% include copy.html %}

如果您提前知道文档的映射并想要避免映射错误（例如，字段名称的拼写错误），则可以设置`dynamic` 参数为`strict`：

```ruby
client.indices.put_mapping(
    index: 'students', 
    body: {
        dynamic: 'strict',
        properties: {
            first_name: { type: 'keyword' },
            last_name: { type: 'keyword' },
            gpa: { type: 'float'},
            grad_year: { type: 'integer'}
        }  
    }
)
```
{% include copy.html %}

使用严格的映射，您可以将文档索引，但您无法用新字段索引文档。例如，用拼写错误为以下文档索引`grad_yea` 字段失败：

```ruby
document = {
    first_name: 'Connor',
    last_name: 'James',
    gpa: 3.93,
    grad_yea: 2021
}
  
client.index(
    index: 'students',
    body: document,
    id: 100,
    refresh: true
)
```
{% include copy.html %}

OpenSearch返回映射错误：

```bash
{"error":{"root_cause":[{"type":"strict_dynamic_mapping_exception","reason":"mapping set to strict, dynamic introduction of [grad_yea] within [_doc] is not allowed"}],"type":"strict_dynamic_mapping_exception","reason":"mapping set to strict, dynamic introduction of [grad_yea] within [_doc] is not allowed"},"status":400}
```

## 索引一个文档

要索引一个文档，请使用`index` 方法：

```ruby
document = {
    first_name: 'Connor',
    last_name: 'James',
    gpa: 3.93,
    grad_year: 2021
}
  
client.index(
    index: 'students',
    body: document,
    id: 100,
    refresh: true
)
```
{% include copy.html %}

## 更新文档

要更新文档，请使用`update` 方法：

```ruby
client.update(index: 'students', 
              id: 100, 
              body: { doc: { gpa: 3.25 } }, 
              refresh: true)
```
{% include copy.html %}

## 删除文档

要删除文档，请使用`delete` 方法：

```ruby
client.delete(
    index: 'students',
    id: 100,
    refresh: true
)
```
{% include copy.html %}

## 批量操作

您可以同时使用`bulk` 方法。操作可能是相同类型的或不同类型的。

您可以使用`bulk` 方法：

```ruby
actions = [
    { index: { _index: 'students', _id: '200' } },
    { first_name: 'James', last_name: 'Rodriguez', gpa: 3.91, grad_year: 2019 },
    { index: { _index: 'students', _id: '300' } },
    { first_name: 'Nikki', last_name: 'Wolf', gpa: 3.87, grad_year: 2020 }
]
client.bulk(body: actions, refresh: true)
```
{% include copy.html %}

您可以删除多个文档，如下所示：

```ruby
# Deleting multiple documents.
actions = [
    { delete: { _index: 'students', _id: 200 } },
    { delete: { _index: 'students', _id: 300 } }
]
client.bulk(body: actions, refresh: true)
```
{% include copy.html %}

使用时，您可以执行不同的操作`bulk` 如下：

```ruby
actions = [
    { index:  { _index: 'students', _id: 100, data: { first_name: 'Paulo', last_name: 'Santos', gpa: 3.29, grad_year: 2022 } } },
    { index:  { _index: 'students', _id: 200, data: { first_name: 'Shirley', last_name: 'Rodriguez', gpa: 3.92, grad_year: 2020 } } },
    { index:  { _index: 'students', _id: 300, data: { first_name: 'Akua', last_name: 'Mansa', gpa: 3.95, grad_year: 2022 } } },
    { index:  { _index: 'students', _id: 400, data: { first_name: 'John', last_name: 'Stiles', gpa: 3.72, grad_year: 2019 } } },
    { index:  { _index: 'students', _id: 500, data: { first_name: 'Li', last_name: 'Juan', gpa: 3.94, grad_year: 2022 } } },
    { index:  { _index: 'students', _id: 600, data: { first_name: 'Richard', last_name: 'Roe', gpa: 3.04, grad_year: 2020 } } },
    { update: { _index: 'students', _id: 100, data: { doc: { gpa: 3.73 } } } },
    { delete: { _index: 'students', _id: 200  } }
]
client.bulk(body: actions, refresh: true)
```
{% include copy.html %}

在上面的示例中，您将数据和标头传递在一起，然后用`data:` 钥匙。

## 搜索文档

要搜索文档，请使用`search` 方法。以下示例搜索一个学生的名字或姓氏为"James." 它使用一个`multi_match` 查询搜索两个字段（`first_name` 和`last_name`），这正在提高`last_name` 与符合符号相关的字段（`last_name^2`）。

```ruby
q = 'James'
query = {
  'size': 5,
  'query': {
    'multi_match': {
      'query': q,
      'fields': ['first_name', 'last_name^2']
    }
  }
}

response = client.search(
  body: query,
  index: 'students'
)
```
{% include copy.html %}

如果您省略了请求主体`search` 方法，您的查询变成了`match_all` 查询并返回索引中的所有文档：

```ruby
client.search(index: 'students')
```
{% include copy.html %}

## 布尔查询

Ruby客户端公开了完整的OpenSearch查询功能。除了使用匹配查询的简单搜索外，您还可以创建一个更复杂的布尔查询，以搜索2022年毕业并按姓氏对其进行排序的学生。在下面的示例中，搜索仅限于10个文档。

```ruby
query = {
    'query': {
        'bool': {
        'filter': {
            'term': {
                'grad_year': 2022
                
            }
        }
        }
    },
    'sort': {
        'last_name': {
            'order': 'asc'
        }
    }       
}

response = client.search(index: 'students', from: 0, size: 10, body: query)
```
{% include copy.html %}

## 并发-搜索

您可以一起进行几个查询，并执行多个查询-使用`msearch` 方法。以下代码搜索GPA的学生不在3.1; 3.9范围内：

```ruby
actions = [
    {},
    {query: {range: {gpa: {gt: 3.9}}}},
    {},
    {query: {range: {gpa: {lt: 3.1}}}}
]
response = client.msearch(index: 'students', body: actions)
```
{% include copy.html %}

## 滚动

您可以使用Scroll API分页搜索结果：

```ruby
response = client.search(index: index_name, scroll: '2m', size: 2)

while response['hits']['hits'].size.positive?
    scroll_id = response['_scroll_id']
    puts(response['hits']['hits'].map { |doc| [doc['_source']['first_name'] + ' ' + doc['_source']['last_name']] })
    response = client.scroll(scroll: '1m', body: { scroll_id: scroll_id })
end
```
{% include copy.html %}

首先，您发出搜索查询，指定`scroll` 和`size` 参数。这`scroll` 参数告诉OpenSearch保持搜索上下文多长时间。在这种情况下，将其设置为两分钟。这`size` 参数指定您要在每个请求中返回多少文档。

对初始搜索查询的响应包含一个`_scroll_id` 您可以使用下一组文档。为此，您使用`scroll` 方法，再次指定`scroll` 参数和通过`_scroll_id` 在身体里。您无需指定查询或索引`scroll` 方法。这`scroll` 方法返回下一组文档和`_scroll_id`。使用最新`_scroll_id` 要求下一批文档时，因为`_scroll_id` 可以在请求之间更改。

## 删除索引

您可以使用`delete` 方法：

```ruby
response = client.indices.delete(index: index_name)
```
{% include copy.html %}

## 样本程序

以下是一个完整的示例程序，该程序说明了前节中描述的所有概念。Ruby客户端的方法将响应返回为Ruby Hashes，很难阅读。要以漂亮的格式显示JSON响应，示例程序使用`MultiJson.dump` 方法。

```ruby
require 'opensearch'

client = OpenSearch::Client.new(host: 'http://localhost:9200')

# Create an index with non-default settings
index_name = 'students'
index_body = {
    'settings': {
      'index': {
        'number_of_shards': 1,
        'number_of_replicas': 2 
      }
    }
  } 

client.indices.create(
    index: index_name,
    body: index_body
)

# Create a mapping
client.indices.put_mapping(
    index: index_name, 
    body: {
        properties: {
            first_name: { type: 'keyword' },
            last_name: { type: 'keyword' }
        }  
    }
)

# Get mappings
response = client.indices.get_mapping(index: index_name)
puts 'Mappings for the students index:'
puts MultiJson.dump(response, pretty: "true")

# Add one document to the index
puts 'Adding one document:'
document = {
    first_name: 'Connor',
    last_name: 'James',
    gpa: 3.93,
    grad_year: 2021
}
id = 100
  
client.index(
    index: index_name,
    body: document,
    id: id,
    refresh: true
)
  
response = client.search(index: index_name)
puts MultiJson.dump(response, pretty: "true")
  
# Update a document
puts 'Updating a document:'
client.update(index: index_name, id: id, body: { doc: { gpa: 3.25 } }, refresh: true)
response = client.search(index: index_name)
puts MultiJson.dump(response, pretty: "true")
print 'The updated gpa is '
puts response['hits']['hits'].map { |doc| doc['_source']['gpa'] }

# Add many documents in bulk
documents = [
{ index: { _index: index_name, _id: '200' } },
{ first_name: 'James', last_name: 'Rodriguez', gpa: 3.91, grad_year: 2019},
{ index: { _index: index_name, _id: '300' } },
{ first_name: 'Nikki', last_name: 'Wolf', gpa: 3.87, grad_year: 2020}
]
client.bulk(body: documents, refresh: true)

# Get all documents in the index
response = client.search(index: index_name)
puts 'All documents in the index after bulk upload:'
puts MultiJson.dump(response, pretty: "true")

# Search for a document using a multi_match query
puts 'Searching for documents that match "James":'
q = 'James'
query = {
  'size': 5,
  'query': {
    'multi_match': {
      'query': q,
      'fields': ['first_name', 'last_name^2']
    }
  }
}

response = client.search(
  body: query,
  index: index_name
)
puts MultiJson.dump(response, pretty: "true")

# Delete the document
response = client.delete(
index: index_name,
id: id,
refresh: true
)

response = client.search(index: index_name)
puts 'Documents in the index after one document was deleted:'
puts MultiJson.dump(response, pretty: "true")

# Delete multiple documents
actions = [
    { delete: { _index: index_name, _id: 200 } },
    { delete: { _index: index_name, _id: 300 } }
]
client.bulk(body: actions, refresh: true)

response = client.search(index: index_name)

puts 'Documents in the index after all documents were deleted:'
puts MultiJson.dump(response, pretty: "true")

# Bulk several operations together
actions = [
    { index:  { _index: index_name, _id: 100, data: { first_name: 'Paulo', last_name: 'Santos', gpa: 3.29, grad_year: 2022 } } },
    { index:  { _index: index_name, _id: 200, data: { first_name: 'Shirley', last_name: 'Rodriguez', gpa: 3.92, grad_year: 2020 } } },
    { index:  { _index: index_name, _id: 300, data: { first_name: 'Akua', last_name: 'Mansa', gpa: 3.95, grad_year: 2022 } } },
    { index:  { _index: index_name, _id: 400, data: { first_name: 'John', last_name: 'Stiles', gpa: 3.72, grad_year: 2019 } } },
    { index:  { _index: index_name, _id: 500, data: { first_name: 'Li', last_name: 'Juan', gpa: 3.94, grad_year: 2022 } } },
    { index:  { _index: index_name, _id: 600, data: { first_name: 'Richard', last_name: 'Roe', gpa: 3.04, grad_year: 2020 } } },
    { update: { _index: index_name, _id: 100, data: { doc: { gpa: 3.73 } } } },
    { delete: { _index: index_name, _id: 200  } }
]
client.bulk(body: actions, refresh: true)

puts 'All documents in the index after bulk operations with scrolling:'
response = client.search(index: index_name, scroll: '2m', size: 2)

while response['hits']['hits'].size.positive?
    scroll_id = response['_scroll_id']
    puts(response['hits']['hits'].map { |doc| [doc['_source']['first_name'] + ' ' + doc['_source']['last_name']] })
    response = client.scroll(scroll: '1m', body: { scroll_id: scroll_id })
end

# Multi search
actions = [
    {},
    {query: {range: {gpa: {gt: 3.9}}}},
    {},
    {query: {range: {gpa: {lt: 3.1}}}}
]
response = client.msearch(index: index_name, body: actions)

puts 'Multi search results:'
puts MultiJson.dump(response, pretty: "true")

# Boolean query
query = {
    'query': {
        'bool': {
        'filter': {
            'term': {
                'grad_year': 2022
                
            }
        }
        }
    },
    'sort': {
        'last_name': {
            'order': 'asc'
        }
    }       
}

response = client.search(index: index_name, from: 0, size: 10, body: query)

puts 'Boolean query search results:'
puts MultiJson.dump(response, pretty: "true")

# Delete the index
puts 'Deleting the index:'
response = client.indices.delete(index: index_name)

puts MultiJson.dump(response, pretty: "true")
```
{% include copy.html %}

# Ruby AWS SIGV4客户

这[OpenSearch-AWS-SIGV4](https://github.com/opensearch-project/opensearch-ruby-aws-sigv4) 宝石提供`OpenSearch::Aws::Sigv4Client` 班级，具有所有功能`OpenSearch::Client`。这两个客户之间的唯一区别是`OpenSearch::Aws::Sigv4Client` 需要一个实例`Aws::Sigv4::Signer` 在实例化期间与AWS进行身份验证：

```ruby
require 'opensearch-aws-sigv4'
require 'aws-sigv4'

signer = Aws::Sigv4::Signer.new(service: 'es',
                                region: 'us-west-2',
                                access_key_id: 'key_id',
                                secret_access_key: 'secret')

client = OpenSearch::Aws::Sigv4Client.new({ log: true }, signer)

client.cluster.health

client.transport.reload_connections!

client.search q: 'test'
```
{％包括copy.html％


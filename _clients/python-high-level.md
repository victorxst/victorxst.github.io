---
layout: default
title: 高级Python客户端
nav_order: 5
---

OpenSearch High-级Python客户端（`opensearch-dsl-py`）将在版本2.1.0之后进行弃用。我们建议切换到[Python客户（`opensearch-py`）]({{site.url}}{{site.baseurl}}/clients/python-low-level/)，现在包括`opensearch-dsl-py`。
{: .warning}

# 高的-python客户端

OpenSearch High-级Python客户端（`opensearch-dsl-py`）为通用搜索实体（例如文档）提供包装类别，因此您可以将其作为python对象合作。此外，高-Level Client简化了为常见OpenSearch操作的方便的Python方法编写查询和供应。高-级别Python客户端支持创建和索引文档，在有或没有过滤器的情况下进行搜索以及使用查询更新文档。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-DSL-PY仓库](https://github.com/opensearch-project/opensearch-dsl-py)。

## 设置

要将客户端添加到您的项目中，请使用[pip](https://pip.pypa.io/)：

```bash
pip install opensearch-dsl
```
{％include copy.html％}

安装客户端后，您可以像其他任何模块一样导入它：

```python
from opensearchpy import OpenSearch
from opensearch_dsl import Search
```
{％include copy.html％}

## 连接到OpenSearch

要连接到默认的OpenSearch主机，请在使用安全插件时使用SSL创建客户对象。您可以将默认凭据用于测试目的：

```python
host = 'localhost'
port = 9200
auth = ('admin', 'admin') # For testing only. Don't store credentials in code.
ca_certs_path = '/full/path/to/root-ca.pem' # Provide a CA bundle if you use intermediate CAs with your root CA.

# Create the client with SSL/TLS enabled, but hostname verification disabled.
client = OpenSearch(
    hosts = [{'host': host, 'port': port}],
    http_compress = True, # enables gzip compression for request bodies
    http_auth = auth,
    use_ssl = True,
    verify_certs = True,
    ssl_assert_hostname = False,
    ssl_show_warn = False,
    ca_certs = ca_certs_path
)
```
{％include copy.html％}

如果您有自己的客户证书，请在`client_cert_path` 和`client_key_path` 参数：

```python
host = 'localhost'
port = 9200
auth = ('admin', 'admin') # For testing only. Don't store credentials in code.
ca_certs_path = '/full/path/to/root-ca.pem' # Provide a CA bundle if you use intermediate CAs with your root CA.

# Optional client certificates if you don't want to use HTTP basic authentication.
client_cert_path = '/full/path/to/client.pem'
client_key_path = '/full/path/to/client-key.pem'

# Create the client with SSL/TLS enabled, but hostname verification disabled.
client = OpenSearch(
    hosts = [{'host': host, 'port': port}],
    http_compress = True, # enables gzip compression for request bodies
    http_auth = auth,
    client_cert = client_cert_path,
    client_key = client_key_path,
    use_ssl = True,
    verify_certs = True,
    ssl_assert_hostname = False,
    ssl_show_warn = False,
    ca_certs = ca_certs_path
)
```
{％include copy.html％}

如果您不使用安全插件，请使用禁用SSL创建客户端对象：

```python
host = 'localhost'
port = 9200

# Create the client with SSL/TLS and hostname verification disabled.
client = OpenSearch(
    hosts = [{'host': host, 'port': port}],
    http_compress = True, # enables gzip compression for request bodies
    use_ssl = False,
    verify_certs = False,
    ssl_assert_hostname = False,
    ssl_show_warn = False
)
```
{％include copy.html％}

## 创建索引

要创建OpenSearch索引，请使用`client.indices.create()` 方法。您可以使用以下代码构建具有自定义设置的JSON对象：

```python
index_name = 'my-dsl-index'
index_body = {
  'settings': {
    'index': {
      'number_of_shards': 4
    }
  }
}

response = client.indices.create(index_name, body=index_body)
```
{％include copy.html％}

## 索引文档

您可以创建一个类来表示您在OpenSearch中您将索引的文档，通过扩展`Document` 班级：

```python
class Movie(Document):
    title = Text(fields={'raw': Keyword()})
    director = Text()
    year = Text()

    class Index:
        name = index_name

    def save(self, ** kwargs):
        return super(Movie, self).save(** kwargs)
```
{％include copy.html％}

要索引文档，请创建一个新类的对象，然后调用其`save()` 方法：

```python
# Set up the opensearch-py version of the document
Movie.init(using=client)
doc = Movie(meta={'id': 1}, title='Moneyball', director='Bennett Miller', year='2011')
response = doc.save(using=client)
```
{％include copy.html％}

## 执行批量操作

您可以同时使用`bulk()` 客户的方法。操作可能是相同类型的或不同类型的。请注意，操作必须由`\n` 整个字符串必须是一行：

```python
movies = '{ "index" : { "_index" : "my-dsl-index", "_id" : "2" } } \n { "title" : "Interstellar", "director" : "Christopher Nolan", "year" : "2014"} \n { "create" : { "_index" : "my-dsl-index", "_id" : "3" } } \n { "title" : "Star Trek Beyond", "director" : "Justin Lin", "year" : "2015"} \n { "update" : {"_id" : "3", "_index" : "my-dsl-index" } } \n { "doc" : {"year" : "2016"} }'

client.bulk(movies)
```
{％include copy.html％}

## 搜索文档

您可以使用`Search` 构建查询的类。以下代码通过过滤器创建布尔查询：

```python
s = Search(using=client, index=index_name) \
    .filter("term", year="2011") \
    .query("match", title="Moneyball")

response = s.execute()
```
{％include copy.html％}

前面的查询等效于OpenSearch域中的以下查询-特定语言（DSL）：

```json
GET my-dsl-index/_search 
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "Moneyball"
        }
      },
      "filter": {
        "term" : {
          "year": 2011
        }
      }
    }
  }
}
```

## 删除文档

您可以使用`client.delete()` 方法：

```python
response = client.delete(
    index = 'my-dsl-index',
    id = '1'
)
```
{％include copy.html％}

## 删除索引

您可以使用`client.indices.delete()` 方法：

```python
response = client.indices.delete(
    index = 'my-dsl-index'
)
```
{％include copy.html％}

## 样本程序

以下示例程序创建了一个客户端，添加了一个非索引-默认设置，插入文档，执行批量操作，搜索文档，删除文档，然后删除索引：

```python
from opensearchpy import OpenSearch
from opensearch_dsl import Search, Document, Text, Keyword

host = 'localhost'
port = 9200

auth = ('admin', 'admin')  # For testing only. Don't store credentials in code.
ca_certs_path = 'root-ca.pem'

# Create the client with SSL/TLS enabled, but hostname verification disabled.
client = OpenSearch(
    hosts=[{'host': host, 'port': port}],
    http_compress=True,  # enables gzip compression for request bodies
    # http_auth=auth,
    use_ssl=False,
    verify_certs=False,
    ssl_assert_hostname=False,
    ssl_show_warn=False,
    # ca_certs=ca_certs_path
)
index_name = 'my-dsl-index'

index_body = {
  'settings': {
    'index': {
      'number_of_shards': 4
    }
  }
}

response = client.indices.create(index_name, index_body)
print('\nCreating index:')
print(response)

# Create the structure of the document
class Movie(Document):
    title = Text(fields={'raw': Keyword()})
    director = Text()
    year = Text()

    class Index:
        name = index_name

    def save(self, ** kwargs):
        return super(Movie, self).save(** kwargs)

# Set up the opensearch-py version of the document
Movie.init(using=client)
doc = Movie(meta={'id': 1}, title='Moneyball', director='Bennett Miller', year='2011')
response = doc.save(using=client)

print('\nAdding document:')
print(response)

# Perform bulk operations

movies = '{ "index" : { "_index" : "my-dsl-index", "_id" : "2" } } \n { "title" : "Interstellar", "director" : "Christopher Nolan", "year" : "2014"} \n { "create" : { "_index" : "my-dsl-index", "_id" : "3" } } \n { "title" : "Star Trek Beyond", "director" : "Justin Lin", "year" : "2015"} \n { "update" : {"_id" : "3", "_index" : "my-dsl-index" } } \n { "doc" : {"year" : "2016"} }'

client.bulk(movies)

# Search for the document.
s = Search(using=client, index=index_name) \
    .filter('term', year='2011') \
    .query('match', title='Moneyball')

response = s.execute()

print('\nSearch results:')
for hit in response:
    print(hit.meta.score, hit.title)
    
# Delete the document.
print('\nDeleting document:')
print(response)

# Delete the index.
response = client.indices.delete(
    index = index_name
)

print('\nDeleting index:')
print(response)
```
{％包括copy.html％


---
layout: default
title: 低级 Python 客户端
nav_order: 10
redirect_from: 
  - /clients/python/
---

# 低级 Python 客户端

opensearch low-级Python客户端（`opensearch-py`）为OpenSearch REST API提供包装方法，以便您可以在Python中更自然地与群集进行交互。您可以为群集创建一个OpenSearch客户端并调用客户端的构建客户端，而不是向给定的URL发送RAW HTTP请求-在功能中。有关客户的完整API文档和其他示例，请参见[`opensearch-py` API文档](https://opensearch-project.github.io/opensearch-py/)。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-PY仓库](https://github.com/opensearch-project/opensearch-py)。

## 设置

要将客户端添加到您的项目中，请使用[pip](https://pip.pypa.io/)：

```bash
pip install opensearch-py
```
{％include copy.html％}

安装客户端后，您可以像其他任何模块一样导入它：

```python
from opensearchpy import OpenSearch
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

## 连接到Amazon OpenSearch服务

下面的示例说明了连接到Amazon OpenSearch服务：

```python
from opensearchpy import OpenSearch, RequestsHttpConnection, AWSV4SignerAuth
import boto3

host = '' # cluster endpoint, for example: my-test-domain.us-east-1.es.amazonaws.com
region = 'us-west-2'
service = 'es'
credentials = boto3.Session().get_credentials()
auth = AWSV4SignerAuth(credentials, region, service)

client = OpenSearch(
    hosts = [{'host': host, 'port': 443}],
    http_auth = auth,
    use_ssl = True,
    verify_certs = True,
    connection_class = RequestsHttpConnection,
    pool_maxsize = 20
)
```
{％include copy.html％}

## 连接到Amazon OpenSearch无服务器

以下示例说明了连接到Amazon OpenSearch无服务器服务：

```python
from opensearchpy import OpenSearch, RequestsHttpConnection, AWSV4SignerAuth
import boto3

host = '' # cluster endpoint, for example: my-test-domain.us-east-1.aoss.amazonaws.com
region = 'us-west-2'
service = 'aoss'
credentials = boto3.Session().get_credentials()
auth = AWSV4SignerAuth(credentials, region, service)

client = OpenSearch(
    hosts = [{'host': host, 'port': 443}],
    http_auth = auth,
    use_ssl = True,
    verify_certs = True,
    connection_class = RequestsHttpConnection,
    pool_maxsize = 20
)
```
{％include copy.html％}


## 创建索引

要创建OpenSearch索引，请使用`client.indices.create()` 方法。您可以使用以下代码构建具有自定义设置的JSON对象：

```python
index_name = 'python-test-index'
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

您可以使用`client.index()` 方法：

```python
document = {
  'title': 'Moneyball',
  'director': 'Bennett Miller',
  'year': '2011'
}

response = client.index(
    index = 'python-test-index',
    body = document,
    id = '1',
    refresh = True
)
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

搜索文档的最简单方法是构建查询字符串。以下代码使用多个-匹配查询以在标题和导演字段中搜索“ Miller”。它增加了标题字段中具有“米勒”的文档：

```python
q = 'miller'
query = {
  'size': 5,
  'query': {
    'multi_match': {
      'query': q,
      'fields': ['title^2', 'director']
    }
  }
}

response = client.search(
    body = query,
    index = 'python-test-index'
)
```
{％include copy.html％}

## 删除文档

您可以使用`client.delete()` 方法：

```python
response = client.delete(
    index = 'python-test-index',
    id = '1'
)
```
{％include copy.html％}

## 删除索引

您可以使用`client.indices.delete()` 方法：

```python
response = client.indices.delete(
    index = 'python-test-index'
)
```
{％include copy.html％}

## 样本程序

以下示例程序创建了一个客户端，添加了一个非索引-默认设置，插入文档，执行批量操作，搜索文档，删除文档，然后删除索引：

```python
from opensearchpy import OpenSearch

host = 'localhost'
port = 9200
auth = ('admin', 'admin') # For testing only. Don't store credentials in code.
ca_certs_path = '/full/path/to/root-ca.pem' # Provide a CA bundle if you use intermediate CAs with your root CA.

# Optional client certificates if you don't want to use HTTP basic authentication.
# client_cert_path = '/full/path/to/client.pem'
# client_key_path = '/full/path/to/client-key.pem'

# Create the client with SSL/TLS enabled, but hostname verification disabled.
client = OpenSearch(
    hosts = [{'host': host, 'port': port}],
    http_compress = True, # enables gzip compression for request bodies
    http_auth = auth,
    # client_cert = client_cert_path,
    # client_key = client_key_path,
    use_ssl = True,
    verify_certs = True,
    ssl_assert_hostname = False,
    ssl_show_warn = False,
    ca_certs = ca_certs_path
)

# Create an index with non-default settings.
index_name = 'python-test-index'
index_body = {
  'settings': {
    'index': {
      'number_of_shards': 4
    }
  }
}

response = client.indices.create(index_name, body=index_body)
print('\nCreating index:')
print(response)

# Add a document to the index.
document = {
  'title': 'Moneyball',
  'director': 'Bennett Miller',
  'year': '2011'
}
id = '1'

response = client.index(
    index = index_name,
    body = document,
    id = id,
    refresh = True
)

print('\nAdding document:')
print(response)

# Perform bulk operations

movies = '{ "index" : { "_index" : "my-dsl-index", "_id" : "2" } } \n { "title" : "Interstellar", "director" : "Christopher Nolan", "year" : "2014"} \n { "create" : { "_index" : "my-dsl-index", "_id" : "3" } } \n { "title" : "Star Trek Beyond", "director" : "Justin Lin", "year" : "2015"} \n { "update" : {"_id" : "3", "_index" : "my-dsl-index" } } \n { "doc" : {"year" : "2016"} }'

client.bulk(movies)

# Search for the document.
q = 'miller'
query = {
  'size': 5,
  'query': {
    'multi_match': {
      'query': q,
      'fields': ['title^2', 'director']
    }
  }
}

response = client.search(
    body = query,
    index = index_name
)
print('\nSearch results:')
print(response)

# Delete the document.
response = client.delete(
    index = index_name,
    id = id
)

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


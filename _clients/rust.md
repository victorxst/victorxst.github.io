---
layout: default
title: Rust客户端
nav_order: 100
---

# Rust客户端

OpenSearch Rust客户端可让您将Rust应用程序与OpenSearch集群中的数据联系起来。有关客户的完整API文档和其他示例，请参见[OpenSearch Docs.rs文档](https://docs.rs/opensearch/)。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-RS回购](https://github.com/opensearch-project/opensearch-rs)。

## 设置

如果您正在开始一个新项目，请添加`opensearch` 箱子到货物.toml：

```rust
[dependencies]
opensearch = "1.0.0"
```
{％include copy.html％}

此外，您可能需要添加以下内容`serde` 有助于将类型序列化的依赖项和json的应对响应：

```rust
serde = "~1"
serde_json = "~1"
```
{％include copy.html％}

生锈客户使用更高的-等级[`reqwest`](https://crates.io/crates/reqwest) HTTP客户库http请求，ReqWest使用[`tokio`](https://crates.io/crates/tokio) 支持异步请求的平台。如果您打算使用异步功能，则需要添加`tokio` 对货物的依赖。

```rust
tokio = { version = "*", features = ["full"] }
```
{％include copy.html％}

看到[样本程序](#sample-program) 完整货物文件的部分。

要使用Rust Client API，请导入您需要的模块，结构和枚举：

```rust
use opensearch::OpenSearch;
```
{％include copy.html％}

## 连接到OpenSearch

要连接到默认的OpenSearch主机，请创建一个默认客户端对象，该对象连接到地址的OpenSearch`http://localhost:9200`：

```rust
let client = OpenSearch::default();
```
{％include copy.html％}

要连接到以不同地址运行的OpenSearch主机，请使用指定地址创建客户端：

```rust
let transport = Transport::single_node("http://localhost:9200")?;
let client = OpenSearch::new(transport);
```
{％include copy.html％}

另外，您可以自定义URL并通过创建一个连接池`TransportBuilder` 结构并将其传递给`OpenSearch::new` 创建客户端的新实例：

```rust
let url = Url::parse("http://localhost:9200")?;
let conn_pool = SingleNodeConnectionPool::new(url);
let transport = TransportBuilder::new(conn_pool).disable_proxy().build()?;
let client = OpenSearch::new(transport);
```
{％include copy.html％}

## 连接到Amazon OpenSearch服务

下面的示例说明了连接到Amazon OpenSearch服务：

```rust
let url = Url::parse("https://...");
let service_name = "es";
let conn_pool = SingleNodeConnectionPool::new(url?);
let region_provider = RegionProviderChain::default_provider().or_else("us-east-1");
let aws_config = aws_config::from_env().region(region_provider).load().await.clone();
let transport = TransportBuilder::new(conn_pool)
    .auth(aws_config.clone().try_into()?)
    .service_name(service_name)
    .build()?;
let client = OpenSearch::new(transport);
```
{％include copy.html％}

## 连接到Amazon OpenSearch无服务器

以下示例说明了连接到Amazon OpenSearch无服务器服务：

```rust
let url = Url::parse("https://...");
let service_name = "aoss";
let conn_pool = SingleNodeConnectionPool::new(url?);
let region_provider = RegionProviderChain::default_provider().or_else("us-east-1");
let aws_config = aws_config::from_env().region(region_provider).load().await.clone();
let transport = TransportBuilder::new(conn_pool)
    .auth(aws_config.clone().try_into()?)
    .service_name(service_name)
    .build()?;
let client = OpenSearch::new(transport);
```
{％include copy.html％}


## 创建索引

要创建OpenSearch索引，请使用`create` 函数`opensearch::indices::Indices` 结构。您可以使用以下代码构建具有自定义映射的JSON对象：

```rust
let response = client
    .indices()
    .create(IndicesCreateParts::Index("movies"))
    .body(json!({
        "mappings" : {
            "properties" : {
                "title" : { "type" : "text" }
            }
        }
    }))
    .send()
    .await?;
```
{％include copy.html％}

## 索引文档

您可以使用客户端的文档进行索引`index` 功能：

```rust
let response = client
    .index(IndexParts::IndexId("movies", "1"))
    .body(json!({
        "id": 1,
        "title": "Moneyball",
        "director": "Bennett Miller",
        "year": "2011"
    }))
    .send()
    .await?;
```
{％include copy.html％}

## 执行批量操作

您可以通过使用客户端同时执行多个操作`bulk` 功能。首先，创建散装API调用的JSON主体，然后将其传递给`bulk` 功能：

```rust
let mut body: Vec<JsonBody<_>> = Vec::with_capacity(4);

// add the first operation and document
body.push(json!({"index": {"_id": "2"}}).into());
body.push(json!({
    "id": 2,
    "title": "Interstellar",
    "director": "Christopher Nolan",
    "year": "2014"
}).into());

// add the second operation and document
body.push(json!({"index": {"_id": "3"}}).into());
body.push(json!({
    "id": 3,
    "title": "Star Trek Beyond",
    "director": "Justin Lin",
    "year": "2015"
}).into());

let response = client
    .bulk(BulkParts::Index("movies"))
    .body(body)
    .send()
    .await?;
```
{％include copy.html％}

## 搜索文档

搜索文档的最简单方法是构建查询字符串。以下代码使用`multi_match` 查询搜索"miller" 在标题和导演领域。它增加了文件"miller" 出现在标题字段中：

```rust
response = client
    .search(SearchParts::Index(&["movies"]))
    .from(0)
    .size(10)
    .body(json!({
        "query": {
            "multi_match": {
                "query": "miller",
                "fields": ["title^2", "director"]
            }           
        }
    }))
    .send()
    .await?;
```
{％include copy.html％}

然后，您可以阅读响应主体，并在JSON上迭代`hits` 数组阅读所有`_source` 文件：

```rust
let response_body = response.json::<Value>().await?;
for hit in response_body["hits"]["hits"].as_array().unwrap() {
    // print the source document
    println!("{}", serde_json::to_string_pretty(&hit["_source"]).unwrap());
}
```
{％include copy.html％}

## 删除文档

您可以使用客户端的文档删除文档`delete` 功能：

```rust
let response = client
    .delete(DeleteParts::IndexId("movies", "2"))
    .send()
    .await?;
```
{％include copy.html％}

## 删除索引

您可以使用`delete` 函数`opensearch::indices::Indices` 结构：

```rust
let response = client
    .indices()
    .delete(IndicesDeleteParts::Index(&["movies"]))
    .send()
    .await?;
```
{％include copy.html％}

## 样本程序

示例程序使用以下货物.toml文件，其中包括[设置](#setup) 部分：

```rust
[package]
name = "os_rust_project"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
opensearch = "1.0.0"
tokio = { version = "*", features = ["full"] }
serde = "~1"
serde_json = "~1"
```
{％include copy.html％}

以下示例程序创建了一个客户端，添加了一个非索引-默认映射，插入文档，执行批量操作，搜索文档，删除文档，然后删除索引：

```rust
use opensearch::{DeleteParts, OpenSearch, IndexParts, http::request::JsonBody, BulkParts, SearchParts};
use opensearch::{indices::{IndicesDeleteParts, IndicesCreateParts}};
use serde_json::{json, Value};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = OpenSearch::default();

    // Create an index
    let mut response = client
    .indices()
    .create(IndicesCreateParts::Index("movies"))
    .body(json!({
        "mappings" : {
            "properties" : {
                "title" : { "type" : "text" }
            }
        }
    }))
    .send()
    .await?;

    let mut successful = response.status_code().is_success();
    
    if successful {
        println!("Successfully created an index");
    }
    else {
        println!("Could not create an index");
    }

    // Index a single document
    println!("Indexing a single document...");
    response = client
    .index(IndexParts::IndexId("movies", "1"))
    .body(json!({
        "id": 1,
        "title": "Moneyball",
        "director": "Bennett Miller",
        "year": "2011"
    }))
    .send()
    .await?;

    successful = response.status_code().is_success();
    
    if successful {
        println!("Successfully indexed a document");
    }
    else {
        println!("Could not index document");
    }

    // Index multiple documents using the bulk operation

    println!("Indexing multiple documents...");

    let mut body: Vec<JsonBody<_>> = Vec::with_capacity(4);

    // add the first operation and document
    body.push(json!({"index": {"_id": "2"}}).into());
    body.push(json!({
        "id": 2,
        "title": "Interstellar",
        "director": "Christopher Nolan",
        "year": "2014"
    }).into());

    // add the second operation and document
    body.push(json!({"index": {"_id": "3"}}).into());
    body.push(json!({
        "id": 3,
        "title": "Star Trek Beyond",
        "director": "Justin Lin",
        "year": "2015"
    }).into());

    response = client
        .bulk(BulkParts::Index("movies"))
        .body(body)
        .send()
        .await?;

    let mut response_body = response.json::<Value>().await?;
    successful = response_body["errors"].as_bool().unwrap() == false;

    if successful {
        println!("Successfully performed bulk operations");
    }
    else {
        println!("Could not perform bulk operations");
    }

    // Search for a document

    println!("Searching for a document...");
    response = client
    .search(SearchParts::Index(&["movies"]))
    .from(0)
    .size(10)
    .body(json!({
        "query": {
            "multi_match": {
                "query": "miller",
                "fields": ["title^2", "director"]
            }           
        }
    }))
    .send()
    .await?;

    response_body = response.json::<Value>().await?;
    for hit in response_body["hits"]["hits"].as_array().unwrap() {
        // print the source document
        println!("{}", serde_json::to_string_pretty(&hit["_source"]).unwrap());
    }

    // Delete a document

    response = client
    .delete(DeleteParts::IndexId("movies", "2"))
    .send()
    .await?;

    successful = response.status_code().is_success();
    
    if successful {
        println!("Successfully deleted a document");
    }
    else {
        println!("Could not delete document");
    }

    // Delete the index

    response = client
    .indices()
    .delete(IndicesDeleteParts::Index(&["movies"]))
    .send()
    .await?;

    successful = response.status_code().is_success();

    if successful {
        println!("Successfully deleted the index");
    }
    else {
        println!("Could not delete the index");
    }
    
    Ok(())
}
```
{％包括copy.html％


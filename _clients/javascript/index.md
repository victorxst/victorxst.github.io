---
layout: default
title: JavaScript客户端
has_children: true
nav_order: 40
redirect_from:
  - /clients/javascript/
---

# JavaScript客户端

OpenSearch JavaScript（JS）客户端提供了一种与OpenSearch集群交互的更安全，更轻松的方法。您可以构建一个opensearch客户端，该客户端将请求将请求发送到群集。有关客户的完整API文档和其他示例，请参见[JS客户端API文档](https://opensearch-project.github.io/opensearch-js/2.2/index.html)。

客户端包含一个API库，可让您在群集上执行不同的操作并返回标准响应主体。这里的示例演示了一些基本操作，例如创建索引，添加文档和搜索数据。

您可以使用辅助方法来简化复杂的API任务的使用。有关更多信息，请参阅[辅助方法]({{site.url}}{{site.baseurl}}/clients/javascript/helpers/)。有关更高级的索引操作，请参阅[`opensearch-js` 向导](https://github.com/opensearch-project/opensearch-js/tree/main/guides) 在github。

## 设置

要将客户端添加到您的项目中，请从[NPM](https://www.npmjs.com)：

```bash
npm install @opensearch-project/opensearch
```
{% include copy.html %}

要安装客户端的特定主要版本，请运行以下命令：

```bash
npm install @opensearch-project/opensearch@<version>
```
{% include copy.html %}

如果您希望手动添加客户端或只想检查源代码，请参见[OpenSearch-JS](https://github.com/opensearch-project/opensearch-js) 在github上。

然后需要客户：

```javascript
const { Client } = require("@opensearch-project/opensearch");
```
{% include copy.html %}

## 连接到OpenSearch

要连接到默认的OpenSearch主机，请使用地址创建客户端对象`https://localhost:9200` 如果您使用的是安全插件：

```javascript
var host = "localhost";
var protocol = "https";
var port = 9200;
var auth = "admin:admin"; // For testing only. Don't store credentials in code.
var ca_certs_path = "/full/path/to/root-ca.pem";

// Optional client certificates if you don't want to use HTTP basic authentication.
// var client_cert_path = '/full/path/to/client.pem'
// var client_key_path = '/full/path/to/client-key.pem'

// Create a client with SSL/TLS enabled.
var { Client } = require("@opensearch-project/opensearch");
var fs = require("fs");
var client = new Client({
  node: protocol + "://" + auth + "@" + host + ":" + port,
  ssl: {
    ca: fs.readFileSync(ca_certs_path),
    // You can turn off certificate verification (rejectUnauthorized: false) if you're using 
    // self-signed certificates with a hostname mismatch.
    // cert: fs.readFileSync(client_cert_path),
    // key: fs.readFileSync(client_key_path)
  },
});
```
{% include copy.html %}

## 使用Amazon OpenSearch服务进行身份验证 -  AWS SIGV4

使用以下代码通过AWS V2 SDK进行身份验证：

```javascript
const AWS = require('aws-sdk'); // V2 SDK.
const { Client } = require('@opensearch-project/opensearch');
const { AwsSigv4Signer } = require('@opensearch-project/opensearch/aws');

const client = new Client({
  ...AwsSigv4Signer({
    region: 'us-west-2',
    service: 'es',
    // Must return a Promise that resolve to an AWS.Credentials object.
    // This function is used to acquire the credentials when the client start and
    // when the credentials are expired.
    // The Client will refresh the Credentials only when they are expired.
    // With AWS SDK V2, Credentials.refreshPromise is used when available to refresh the credentials.

    // Example with AWS SDK V2:
    getCredentials: () =>
      new Promise((resolve, reject) => {
        // Any other method to acquire a new Credentials object can be used.
        AWS.config.getCredentials((err, credentials) => {
          if (err) {
            reject(err);
          } else {
            resolve(credentials);
          }
        });
      }),
  }),
  node: 'https://search-xxx.region.es.amazonaws.com', // OpenSearch domain URL
});
```
{% include copy.html %}

AWS V2 SDK用于Amazon OpenSearch无服务器

```javascript
const AWS = require('aws-sdk'); // V2 SDK.
const { Client } = require('@opensearch-project/opensearch');
const { AwsSigv4Signer } = require('@opensearch-project/opensearch/aws');

const client = new Client({
  ...AwsSigv4Signer({
    region: 'us-west-2',
    service: 'aoss',
    // Must return a Promise that resolve to an AWS.Credentials object.
    // This function is used to acquire the credentials when the client start and
    // when the credentials are expired.
    // The Client will refresh the Credentials only when they are expired.
    // With AWS SDK V2, Credentials.refreshPromise is used when available to refresh the credentials.

    // Example with AWS SDK V2:
    getCredentials: () =>
      new Promise((resolve, reject) => {
        // Any other method to acquire a new Credentials object can be used.
        AWS.config.getCredentials((err, credentials) => {
          if (err) {
            reject(err);
          } else {
            resolve(credentials);
          }
        });
      }),
  }),
  node: "https://xxx.region.aoss.amazonaws.com" // OpenSearch domain URL
});
```
{% include copy.html %}

使用以下代码通过AWS V3 SDK进行身份验证：

```javascript
const { defaultProvider } = require('@aws-sdk/credential-provider-node'); // V3 SDK.
const { Client } = require('@opensearch-project/opensearch');
const { AwsSigv4Signer } = require('@opensearch-project/opensearch/aws');

const client = new Client({
  ...AwsSigv4Signer({
    region: 'us-east-1',
    service: 'es',  // 'aoss' for OpenSearch Serverless
    // Must return a Promise that resolve to an AWS.Credentials object.
    // This function is used to acquire the credentials when the client start and
    // when the credentials are expired.
    // The Client will refresh the Credentials only when they are expired.
    // With AWS SDK V2, Credentials.refreshPromise is used when available to refresh the credentials.

    // Example with AWS SDK V3:
    getCredentials: () => {
      // Any other method to acquire a new Credentials object can be used.
      const credentialsProvider = defaultProvider();
      return credentialsProvider();
    },
  }),
  node: 'https://search-xxx.region.es.amazonaws.com', // OpenSearch domain URL
  // node: "https://xxx.region.aoss.amazonaws.com" for OpenSearch Serverless
});
```
{% include copy.html %}

AWS V3 SDK Amazon OpenSearch无服务器

```javascript
const { defaultProvider } = require('@aws-sdk/credential-provider-node'); // V3 SDK.
const { Client } = require('@opensearch-project/opensearch');
const { AwsSigv4Signer } = require('@opensearch-project/opensearch/aws');

const client = new Client({
  ...AwsSigv4Signer({
    region: 'us-east-1',
    service: 'aoss',
    // Must return a Promise that resolve to an AWS.Credentials object.
    // This function is used to acquire the credentials when the client start and
    // when the credentials are expired.
    // The Client will refresh the Credentials only when they are expired.
    // With AWS SDK V2, Credentials.refreshPromise is used when available to refresh the credentials.

    // Example with AWS SDK V3:
    getCredentials: () => {
      // Any other method to acquire a new Credentials object can be used.
      const credentialsProvider = defaultProvider();
      return credentialsProvider();
    },
  }),
  node: "https://xxx.region.aoss.amazonaws.com" // OpenSearch domain URL
});
```
{% include copy.html %}

## 创建索引

要创建OpenSearch索引，请使用`indices.create()` 方法。您可以使用以下代码构建具有自定义设置的JSON对象：

```javascript
var index_name = "books";

var settings = {
  settings: {
    index: {
      number_of_shards: 4,
      number_of_replicas: 3,
    },
  },
};

var response = await client.indices.create({
  index: index_name,
  body: settings,
});
```
{% include copy.html %}

## 索引文档

您可以使用客户端的文档进行索引`index` 方法：

```javascript
var document = {
  title: "The Outsider",
  author: "Stephen King",
  year: "2018",
  genre: "Crime fiction",
};

var id = "1";

var response = await client.index({
  id: id,
  index: index_name,
  body: document,
  refresh: true,
});
```
{% include copy.html %}

## 搜索文档

搜索文档的最简单方法是构建查询字符串。以下代码使用`match` 查询搜索"The Outsider" 在标题字段中：

```javascript
var query = {
  query: {
    match: {
      title: {
        query: "The Outsider",
      },
    },
  },
};

var response = await client.search({
  index: index_name,
  body: query,
});
```
{% include copy.html %}

## 删除文档

您可以使用客户端的文档删除文档`delete` 方法：

```javascript
var response = await client.delete({
  index: index_name,
  id: id,
});
```
{% include copy.html %}

## 删除索引

您可以使用`indices.delete()` 方法：

```javascript
var response = await client.indices.delete({
  index: index_name,
});
```
{% include copy.html %}

## 样本程序

以下示例程序创建了一个客户端，添加了一个非索引-默认设置，插入文档，搜索文档，删除文档，然后删除索引：

```javascript
"use strict";

var host = "localhost";
var protocol = "https";
var port = 9200;
var auth = "admin:admin"; // For testing only. Don't store credentials in code.
var ca_certs_path = "/full/path/to/root-ca.pem";

// Optional client certificates if you don't want to use HTTP basic authentication.
// var client_cert_path = '/full/path/to/client.pem'
// var client_key_path = '/full/path/to/client-key.pem'

// Create a client with SSL/TLS enabled.
var { Client } = require("@opensearch-project/opensearch");
var fs = require("fs");
var client = new Client({
  node: protocol + "://" + auth + "@" + host + ":" + port,
  ssl: {
    ca: fs.readFileSync(ca_certs_path),
    // You can turn off certificate verification (rejectUnauthorized: false) if you're using 
    // self-signed certificates with a hostname mismatch.
    // cert: fs.readFileSync(client_cert_path),
    // key: fs.readFileSync(client_key_path)
  },
});

async function search() {
  // Create an index with non-default settings.
  var index_name = "books";
  
  var settings = {
    settings: {
      index: {
        number_of_shards: 4,
        number_of_replicas: 3,
      },
    },
  };

  var response = await client.indices.create({
    index: index_name,
    body: settings,
  });

  console.log("Creating index:");
  console.log(response.body);

  // Add a document to the index.
  var document = {
    title: "The Outsider",
    author: "Stephen King",
    year: "2018",
    genre: "Crime fiction",
  };

  var id = "1";

  var response = await client.index({
    id: id,
    index: index_name,
    body: document,
    refresh: true,
  });

  console.log("Adding document:");
  console.log(response.body);

  // Search for the document.
  var query = {
    query: {
      match: {
        title: {
          query: "The Outsider",
        },
      },
    },
  };

  var response = await client.search({
    index: index_name,
    body: query,
  });

  console.log("Search results:");
  console.log(response.body.hits);

  // Delete the document.
  var response = await client.delete({
    index: index_name,
    id: id,
  });

  console.log("Deleting document:");
  console.log(response.body);

  // Delete the index.
  var response = await client.indices.delete({
    index: index_name,
  });

  console.log("Deleting index:");
  console.log(response.body);
}

search().catch(console.log);
```
{% include copy.html %}
## 断路器

这`memoryCircuitBreaker` 选项可用于防止响应有效载荷太大而无法适应客户可用的堆内存的错误。

这`memoryCircuitBreaker` 对象包含两个字段：

- `enabled`：用于打开或关闭断路器的布尔人。默认为`false`。
- `maxPercentage`：决定断路器是否参与的阈值。有效值是[0，1]范围内的浮子，该范围代表小数形式的百分比。超过该范围的任何值都将纠正到`1.0`。

以下示例实例化了启用断路器的客户端，其阈值设置为可用堆尺寸限制的80％：

```javascript
var client = new Client({
  memoryCircuitBreaker: {
    enabled: true,
    maxPercentage: 0.8,
  },
});
```
{％包括copy.html％


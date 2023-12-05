---
layout: default
title: GO客户端
nav_order: 50
---

# GO客户端

OpenSearch GO客户端可让您将GO应用程序与OpenSearch集群中的数据联系起来。该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户的完整API文档和其他示例，请参见[获取客户端API文档](https://pkg.go.dev/github.com/opensearch-project/opensearch-go/v2)。

有关客户端源代码，请参阅[OpenSearch-去回购](https://github.com/opensearch-project/opensearch-go)。


## 设置

如果您要启动一个新项目，请通过运行以下命令来创建一个新模块：

```go
go mod init <mymodulename>
```
{％include copy.html％}

要将GO客户端添加到您的项目中，请像其他任何模块一样导入它：

```go
go get github.com/opensearch-project/opensearch-go
```
{％include copy.html％}

## 连接到OpenSearch

要连接到默认的OpenSearch主机，请使用地址创建客户端对象`https://localhost:9200` 如果您使用的是安全插件：

```go
client, err := opensearch.NewClient(opensearch.Config{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        },
        Addresses: []string{"https://localhost:9200"},
        Username:  "admin", // For testing only. Don't store credentials in code.
        Password:  "admin",
    })
```
{％include copy.html％}

如果您不使用安全插件，请使用地址创建客户端对象`http://localhost:9200`：

```go
client, err := opensearch.NewClient(opensearch.Config{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        },
        Addresses: []string{"http://localhost:9200"},
    })
```
{％include copy.html％}

## 连接到Amazon OpenSearch服务

下面的示例说明了连接到Amazon OpenSearch服务：

```go
package main

import (
	"context"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	opensearch "github.com/opensearch-project/opensearch-go/v2"
	opensearchapi "github.com/opensearch-project/opensearch-go/v2/opensearchapi"
	requestsigner "github.com/opensearch-project/opensearch-go/v2/signer/awsv2"
)

const endpoint = "" // e.g. https://opensearch-domain.region.com or Amazon OpenSearch Serverless endpoint

func main() {
	ctx := context.Background()

	awsCfg, err := config.LoadDefaultConfig(ctx,
		config.WithRegion("<AWS_REGION>"),
		config.WithCredentialsProvider(
			getCredentialProvider("<AWS_ACCESS_KEY>", "<AWS_SECRET_ACCESS_KEY>", "<AWS_SESSION_TOKEN>"),
		),
	)
	if err != nil {
		log.Fatal(err) // Do not log.fatal in a production ready app.
	}

	// Create an AWS request Signer and load AWS configuration using default config folder or env vars.
	signer, err := requestsigner.NewSignerWithService(awsCfg, "es")
	if err != nil {
		log.Fatal(err) // Do not log.fatal in a production ready app.
	}

	// Create an opensearch client and use the request-signer
	client, err := opensearch.NewClient(opensearch.Config{
		Addresses: []string{endpoint},
		Signer:    signer,
	})
	if err != nil {
		log.Fatal("client creation err", err)
	}
}

func getCredentialProvider(accessKey, secretAccessKey, token string) aws.CredentialsProviderFunc {
	return func(ctx context.Context) (aws.Credentials, error) {
		c := &aws.Credentials{
			AccessKeyID:     accessKey,
			SecretAccessKey: secretAccessKey,
			SessionToken:    token,
		}
		return *c, nil
	}
}
```
{％include copy.html％}

## 连接到Amazon OpenSearch无服务器

以下示例说明了连接到Amazon OpenSearch无服务器服务：

```go
package main

import (
	"context"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	opensearch "github.com/opensearch-project/opensearch-go/v2"
	opensearchapi "github.com/opensearch-project/opensearch-go/v2/opensearchapi"
	requestsigner "github.com/opensearch-project/opensearch-go/v2/signer/awsv2"
)

const endpoint = "" // e.g. https://opensearch-domain.region.com or Amazon OpenSearch Serverless endpoint

func main() {
	ctx := context.Background()

	awsCfg, err := config.LoadDefaultConfig(ctx,
		config.WithRegion("<AWS_REGION>"),
		config.WithCredentialsProvider(
			getCredentialProvider("<AWS_ACCESS_KEY>", "<AWS_SECRET_ACCESS_KEY>", "<AWS_SESSION_TOKEN>"),
		),
	)
	if err != nil {
		log.Fatal(err) // Do not log.fatal in a production ready app.
	}

	// Create an AWS request Signer and load AWS configuration using default config folder or env vars.
	signer, err := requestsigner.NewSignerWithService(awsCfg, "aoss")
	if err != nil {
		log.Fatal(err) // Do not log.fatal in a production ready app.
	}

	// Create an opensearch client and use the request-signer
	client, err := opensearch.NewClient(opensearch.Config{
		Addresses: []string{endpoint},
		Signer:    signer,
	})
	if err != nil {
		log.Fatal("client creation err", err)
	}
}

func getCredentialProvider(accessKey, secretAccessKey, token string) aws.CredentialsProviderFunc {
	return func(ctx context.Context) (aws.Credentials, error) {
		c := &aws.Credentials{
			AccessKeyID:     accessKey,
			SecretAccessKey: secretAccessKey,
			SessionToken:    token,
		}
		return *c, nil
	}
}
```
{％include copy.html％}

Go客户端构造函数采用`opensearch.Config{}` 类型，可以使用选项（例如OpenSearch节点地址列表或用户名和密码组合）进行自定义。

要连接到多个OpenSearch节点，请在`Addresses` 范围：

```go
var (
    urls = []string{"http://localhost:9200", "http://localhost:9201", "http://localhost:9202"}
)

client, err := opensearch.NewClient(opensearch.Config{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        },
        Addresses: urls,
})
```
{％include copy.html％}

默认情况下，GO客户端重试的请求最多三次。要自定义重试的数量，请设置`MaxRetries` 范围。此外，您可以更改通过设置请求的响应代码列表`RetryOnStatus` 范围。以下代码段使用自定义创建一个新的GO客户端`MaxRetries` 和`RetryOnStatus` 值：

```go
client, err := opensearch.NewClient(opensearch.Config{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        },
        Addresses: []string{"http://localhost:9200"},
        MaxRetries: 5,
        RetryOnStatus: []int{502, 503, 504},
    })
```
{％include copy.html％}

## 创建索引

要创建OpenSearch索引，请使用`IndicesCreateRequest` 方法。您可以使用以下代码构建具有自定义设置的JSON对象：

```go
settings := strings.NewReader(`{
    'settings': {
        'index': {
            'number_of_shards': 1,
            'number_of_replicas': 0
            }
        }
    }`)

res := opensearchapi.IndicesCreateRequest{
    Index: "go-test-index1", 
    Body:  settings,
}
```
{％include copy.html％}

## 索引文档

您可以使用`IndexRequest` 方法：

```go
document := strings.NewReader(`{
    "title": "Moneyball",
    "director": "Bennett Miller",
    "year": "2011"
}`)

docId := "1"
req := opensearchapi.IndexRequest{
    Index:      "go-test-index1",
    DocumentID: docId,
    Body:       document,
}
insertResponse, err := req.Do(context.Background(), client)
```
{％include copy.html％}

## 执行批量操作

您可以同时使用`Bulk` 客户的方法。操作可能是相同类型的或不同类型的。

```go
blk, err := client.Bulk(
		strings.NewReader(`
    { "index" : { "_index" : "go-test-index1", "_id" : "2" } }
    { "title" : "Interstellar", "director" : "Christopher Nolan", "year" : "2014"}
    { "create" : { "_index" : "go-test-index1", "_id" : "3" } }
    { "title" : "Star Trek Beyond", "director" : "Justin Lin", "year" : "2015"}
    { "update" : {"_id" : "3", "_index" : "go-test-index1" } }
    { "doc" : {"year" : "2016"} }
`),
	)
```
{％include copy.html％}

## 搜索文档

搜索文档的最简单方法是构建查询字符串。以下代码使用`multi_match` 查询搜索"miller" 在标题和导演领域。它增加了文件"miller" 出现在标题字段中：

```go
content := strings.NewReader(`{
    "size": 5,
    "query": {
        "multi_match": {
        "query": "miller",
        "fields": ["title^2", "director"]
        }
    }
}`)

search := opensearchapi.SearchRequest{
    Index: []string{"go-test-index1"},
    Body: content,
}

searchResponse, err := search.Do(context.Background(), client)
```
{％include copy.html％}

## 删除文档

您可以使用`DeleteRequest` 方法：

```go
delete := opensearchapi.DeleteRequest{
    Index:      "go-test-index1",
    DocumentID: "1",
}

deleteResponse, err := delete.Do(context.Background(), client)
```
{％include copy.html％}

## 删除索引

您可以使用`IndicesDeleteRequest` 方法：

```go
deleteIndex := opensearchapi.IndicesDeleteRequest{
    Index: []string{"go-test-index1"},
}

deleteIndexResponse, err := deleteIndex.Do(context.Background(), client)
```
{％include copy.html％}

## 样本程序

以下示例程序创建了一个客户端，添加了一个非索引-默认设置，插入文档，执行批量操作，搜索文档，删除文档，然后删除索引：

```go
package main
import (
    "os"
    "context"
    "crypto/tls"
    "fmt"
    opensearch "github.com/opensearch-project/opensearch-go"
    opensearchapi "github.com/opensearch-project/opensearch-go/opensearchapi"
    "net/http"
    "strings"
)
const IndexName = "go-test-index1"
func main() {
    // Initialize the client with SSL/TLS enabled.
    client, err := opensearch.NewClient(opensearch.Config{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        },
        Addresses: []string{"https://localhost:9200"},
        Username:  "admin", // For testing only. Don't store credentials in code.
        Password:  "admin",
    })
    if err != nil {
        fmt.Println("cannot initialize", err)
        os.Exit(1)
    }

    // Print OpenSearch version information on console.
    fmt.Println(client.Info())

    // Define index settings.
    settings := strings.NewReader(`{
     'settings': {
       'index': {
            'number_of_shards': 1,
            'number_of_replicas': 2
            }
          }
     }`)

    // Create an index with non-default settings.
    res := opensearchapi.IndicesCreateRequest{
        Index: IndexName, 
        Body:  settings,
    }
    fmt.Println("Creating index")
    fmt.Println(res)

    // Add a document to the index.
    document := strings.NewReader(`{
        "title": "Moneyball",
        "director": "Bennett Miller",
        "year": "2011"
    }`)

    docId := "1"
    req := opensearchapi.IndexRequest{
        Index:      IndexName,
        DocumentID: docId,
        Body:       document,
    }
    insertResponse, err := req.Do(context.Background(), client)
    if err != nil {
        fmt.Println("failed to insert document ", err)
        os.Exit(1)
    }
    fmt.Println("Inserting a document")
    fmt.Println(insertResponse)
    defer insertResponse.Body.Close()
   
    // Perform bulk operations.
    blk, err := client.Bulk(
		strings.NewReader(`
    { "index" : { "_index" : "go-test-index1", "_id" : "2" } }
    { "title" : "Interstellar", "director" : "Christopher Nolan", "year" : "2014"}
    { "create" : { "_index" : "go-test-index1", "_id" : "3" } }
    { "title" : "Star Trek Beyond", "director" : "Justin Lin", "year" : "2015"}
    { "update" : {"_id" : "3", "_index" : "go-test-index1" } }
    { "doc" : {"year" : "2016"} }
`),
	)

    if err != nil {
        fmt.Println("failed to perform bulk operations", err)
        os.Exit(1)
    }
    fmt.Println("Performing bulk operations")
    fmt.Println(blk)

    // Search for the document.
    content := strings.NewReader(`{
       "size": 5,
       "query": {
           "multi_match": {
           "query": "miller",
           "fields": ["title^2", "director"]
           }
      }
    }`)

    search := opensearchapi.SearchRequest{
        Index: []string{IndexName},
        Body: content,
    }

    searchResponse, err := search.Do(context.Background(), client)
    if err != nil {
        fmt.Println("failed to search document ", err)
        os.Exit(1)
    }
    fmt.Println("Searching for a document")
    fmt.Println(searchResponse)
    defer searchResponse.Body.Close()

    // Delete the document.
    delete := opensearchapi.DeleteRequest{
        Index:      IndexName,
        DocumentID: docId,
    }

    deleteResponse, err := delete.Do(context.Background(), client)
    if err != nil {
        fmt.Println("failed to delete document ", err)
        os.Exit(1)
    }
    fmt.Println("Deleting a document")
    fmt.Println(deleteResponse)
    defer deleteResponse.Body.Close()

    // Delete the previously created index.
    deleteIndex := opensearchapi.IndicesDeleteRequest{
        Index: []string{IndexName},
    }

    deleteIndexResponse, err := deleteIndex.Do(context.Background(), client)
    if err != nil {
        fmt.Println("failed to delete index ", err)
        os.Exit(1)
    }
    fmt.Println("Deleting the index")
    fmt.Println(deleteIndexResponse)
    defer deleteIndexResponse.Body.Close()
}
```
{％包括copy.html％


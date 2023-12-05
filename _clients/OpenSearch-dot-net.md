---
layout: default
title: 低层.NET客户端
nav_order: 30
has_children: false
parent: .NET客户端
---

# 低层.NET客户端(OpenSearch.Net)

opensearch.net很低-Level .NET客户端，可通过OpenSearch提供通信的基础层。它是无依赖性的，它可以处理圆-罗宾负载平衡，运输和基本请求/响应周期。OpenSearch.net包含所有OpenSearch API端点作为方法。使用OpenSearch.net时，您需要自己构建查询。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-净仓库](https://github.com/opensearch-project/opensearch-net)。

## 稳定版本

本文档反映了该文档中可用的最新更新[GitHub存储库](https://github.com/opensearch-project/opensearch-net) 并可能包括当前稳定版本中无法使用的更改。Nuget当前的稳定版本是[1.2.0](https://www.nuget.org/packages/OpenSearch.Net.Auth.AwsSigV4/1.2.0)。

## 例子

以下示例说明了连接到OpenSearch，索引文档并在数据上发送查询。它使用学生班来代表一个学生，这等同于索引中的一个文档。

```cs
public class Student
{
    public int Id { get; init; }
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public int GradYear { get; init; }
    public double Gpa { get; init; }
}
```
{% include copy.html %}

## 安装opensearch.net客户端

要安装opensearch.net，请下载[opensearch.net nuget软件包](https://www.nuget.org/packages/OpenSearch.Net) 并将其添加到您的项目中。在Microsoft Visual Studio中，请执行以下步骤：
- 在里面**解决方案资源管理器** 面板，对-单击您的解决方案或项目并选择**管理Nuget软件包用于解决方案**。
- 搜索opensearch.net nuget软件包，然后选择**安装**。

另外，您可以将OpenSearch.net添加到.csproj文件：

```xml
<Project>
  ...
  <ItemGroup>
    <PackageReference Include="Opensearch.Net" Version="1.0.0" />
  </ItemGroup>
</Project>
```
{% include copy.html %}

## 连接到OpenSearch

创建OpenSearchLowlevelClient对象时，请使用默认构造函数连接到默认的OpenSearch Host（`http://localhost:9200`）。

```cs
var client  = new OpenSearchLowLevelClient();
```
{% include copy.html %}

要通过带有已知地址的单个节点连接到OpenSearch cluster，请使用该地址创建一个连接configuration对象，然后将其传递到OpenSearch.net constructor：

```cs
var nodeAddress = new Uri("http://myserver:9200");
var config = new ConnectionConfiguration(nodeAddress);
var client = new OpenSearchLowLevelClient(config);
```
{% include copy.html %}

您也可以使用[连接池]({{site.url}}{{site.baseurl}}/clients/dot-net-conventions#connection-pools) 管理集群中的节点。此外，您可以设置连接配置，以使OpenSearch返回响应为格式的JSON。

```cs
var uri = new Uri("http://localhost:9200");
var connectionPool = new SingleNodeConnectionPool(uri);
var settings = new ConnectionConfiguration(connectionPool).PrettyJson();
var client = new OpenSearchLowLevelClient(settings);
```
{% include copy.html %}

要使用多个节点连接到OpenSearch集群，请创建一个带有其地址的连接池。在此示例中，[`SniffingConnectionPool`]({{site.url}}{{site.baseurl}}/clients/dot-net-conventions#connection-pools) 之所以使用，是因为它可以跟踪被删除或添加到群集中的节点，因此它最适合自动扩展的群集。

```cs
var uris = new[]
{
    new Uri("http://localhost:9200"),
    new Uri("http://localhost:9201"),
    new Uri("http://localhost:9202")
};
var connectionPool = new SniffingConnectionPool(uris);
var settings = new ConnectionConfiguration(connectionPool).PrettyJson();
var client = new OpenSearchLowLevelClient(settings);
```
{% include copy.html %}

## 连接到Amazon OpenSearch服务

下面的示例说明了连接到Amazon OpenSearch服务：

```cs
using OpenSearch.Client;
using OpenSearch.Net.Auth.AwsSigV4;

namespace Application
{
    class Program
    {
        static void Main(string[] args)
        {
            var endpoint = new Uri("https://search-xxx.region.es.amazonaws.com");
            var connection = new AwsSigV4HttpConnection(RegionEndpoint.APSoutheast2, service: AwsSigV4HttpConnection.OpenSearchService);
            var config = new ConnectionSettings(endpoint, connection);
            var client = new OpenSearchClient(config);

            Console.WriteLine($"{client.RootNodeInfo().Version.Distribution}: {client.RootNodeInfo().Version.Number}");
        }
    }
}
```
{% include copy.html %}

## 连接到Amazon OpenSearch无服务器

以下示例说明了连接到Amazon OpenSearch无服务器服务：

```cs
using OpenSearch.Client;
using OpenSearch.Net.Auth.AwsSigV4;

namespace Application
{
    class Program
    {
        static void Main(string[] args)
        {
            var endpoint = new Uri("https://search-xxx.region.aoss.amazonaws.com");
            var connection = new AwsSigV4HttpConnection(RegionEndpoint.APSoutheast2, service: AwsSigV4HttpConnection.OpenSearchServerlessService);
            var config = new ConnectionSettings(endpoint, connection);
            var client = new OpenSearchClient(config);

            Console.WriteLine($"{client.RootNodeInfo().Version.Distribution}: {client.RootNodeInfo().Version.Number}");
        }
    }
}
```
{% include copy.html %}


## 使用Connectionsettings

`ConnectionConfiguration` 用于将配置选项传递给opensearch.net客户端。`ConnectionSettings` 继承`ConnectionConfiguration` 并提供其他配置选项。
以下示例使用`ConnectionSettings` 到：
- 设置未指定索引名称的请求的默认索引名称。
- 启用Gzip-压缩请求和响应。
- 信号向OpenSearch返回格式的JSON。
- 使字段名称小写。

```cs
var uri = new Uri("http://localhost:9200");
var connectionPool = new SingleNodeConnectionPool(uri);
var settings = new ConnectionSettings(connectionPool)
    .DefaultIndex("students")
    .EnableHttpCompression()
    .PrettyJson()
    .DefaultFieldNameInferrer(f => f.ToLower());

var client = new OpenSearchLowLevelClient(settings);
```
{% include copy.html %}

## 索引一个文档

要索引文档，您首先需要创建学生班的实例：

```cs
var student = new Student { 
    Id = 100, 
    FirstName = "Paulo", 
    LastName = "Santos", 
    Gpa = 3.93, 
    GradYear = 2021 
};
```
{% include copy.html %}

另外，您可以使用匿名类型创建学生的实例：

```cs
var student = new { 
    Id = 100, 
    FirstName = "Paulo", 
    LastName = "Santos", 
    Gpa = 3.93, 
    GradYear = 2021 
};
```
{% include copy.html %}

接下来，将这个学生上传到`students` 使用`Index` 方法：

```cs
var response = client.Index<StringResponse>("students", "100", 
                                PostData.Serializable(student));
Console.WriteLine(response.Body);
```
{% include copy.html %}

的通用类型参数`Index` 方法指定响应身体类型。在上面的示例中，响应是字符串。

## 使用批量API索引许多文档

要索引许多文档，请使用批量API将许多操作捆绑到一个请求中：

```cs
var studentArray = new object[]
{
    new {index = new { _index = "students", _type = "_doc", _id = "200"}},
    new {   Id = 200, 
            FirstName = "Shirley", 
            LastName = "Rodriguez", 
            Gpa = 3.91, 
            GradYear = 2019
    },
    new {index = new { _index = "students", _type = "_doc", _id = "300"}},
    new {   Id = 300, 
            FirstName = "Nikki", 
            LastName = "Wolf", 
            Gpa = 3.87, 
            GradYear = 2020
    }
};

var manyResponse = client.Bulk<StringResponse>(PostData.MultiJson(studentArray));
```
{% include copy.html %}

您可以将请求主体作为一个匿名对象，字符串，字节数组或流中的流式传输。对于使用Multiline JSON的API，您可以将其发送为字节列表或对象列表，例如上面的示例。这`PostData` 班级具有静态方法，可以以所有这些形式发送身体。

## 搜索文档

要构建查询DSL查询，请在请求主体中使用匿名类型。以下查询搜索所有2021年毕业的学生：

```cs
var searchResponseLow = client.Search<StringResponse>("students",
    PostData.Serializable(
    new
    {
        from = 0,
        size = 20,

        query = new
        {
            term = new
            {
                gradYear = new
                {
                    value = 2019
                }
            }
        }
    })); 

Console.WriteLine(searchResponseLow.Body);
```
{% include copy.html %}

另外，您可以使用字符串来构建请求。使用字符串时，您必须逃脱`"` 特点：

```cs
var searchResponse = client.Search<StringResponse>("students",
    @" {
    ""query"":
        {
            ""match"":
            {
                ""lastName"":
                {
                    ""query"": ""Santos""
                }
            }
        }
    }");

Console.WriteLine(searchResponse.Body);
```
{% include copy.html %}

## 使用opensearch.net方法异步

对于需要异步代码的应用程序，OpenSearch中的所有方法调用都具有异步对应物：

```cs
// synchronous method
var response = client.Index<StringResponse>("students", "100", 
                                PostData.Serializable(student));

// asynchronous method
var response = client.IndexAsync<StringResponse>("students", "100", 
                                    PostData.Serializable(student));
```
{% include copy.html %}

## 处理例外

默认情况下，当操作不成功时，OpenSearch.net不会引发异常。特别是，如果响应状态代码具有此请求的预期值之一，则OpenSearch.net不会引发异常。例如，以下查询搜索不存在的索引中的文档：

```cs
var searchResponse = client.Search<StringResponse>("students1",
    @" {
    ""query"":
        {
            ""match"":
            {
                ""lastName"":
                {
                    ""query"": ""Santos""
                }
            }
        }
    }");

Console.WriteLine(searchResponse.Body);
```
{% include copy.html %}

响应包含错误状态代码404，这是搜索请求的预期错误代码之一，因此没有例外。您可以在`status` 场地：

```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index [students1]",
        "index" : "students1",
        "resource.id" : "students1",
        "resource.type" : "index_or_alias",
        "index_uuid" : "_na_"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index [students1]",
    "index" : "students1",
    "resource.id" : "students1",
    "resource.type" : "index_or_alias",
    "index_uuid" : "_na_"
  },
  "status" : 404
}
```

要配置opensearch.net以抛出异常，请打开`ThrowExceptions()` 设置`ConnectionConfiguration`：

```cs
var uri = new Uri("http://localhost:9200");
var connectionPool = new SingleNodeConnectionPool(uri);
var settings = new ConnectionConfiguration(connectionPool)
                        .PrettyJson().ThrowExceptions();
var client = new OpenSearchLowLevelClient(settings);
```
{% include copy.html %}

您可以使用响应对象的以下属性来确定响应成功：

```cs
Console.WriteLine("Success: " + searchResponse.Success);
Console.WriteLine("SuccessOrKnownError: " + searchResponse.SuccessOrKnownError);
Console.WriteLine("Original Exception: " + searchResponse.OriginalException);
```

- `Success` 如果响应代码在2xx范围内，则返回true或响应代码具有此请求的预期值之一。
- `SuccessOrKnownError` 如果响应成功或响应代码在400-501或505–599范围内，则返回为TRUE。如果Successorknownerror是正确的，则请求未重述。
- `OriginalException` 对不成功的响应持有原始例外。

## 样本程序

以下程序创建索引，索引数据并搜索文档。

```cs
using OpenSearch.Net;
using OpenSearch.Client;

namespace NetClientProgram;

internal class Program
{
    public static void Main(string[] args)
    {
        // Create a client with custom settings
        var uri = new Uri("http://localhost:9200");
        var connectionPool = new SingleNodeConnectionPool(uri);
        var settings = new ConnectionSettings(connectionPool)
            .PrettyJson();
        var client = new OpenSearchLowLevelClient(settings);


        Console.WriteLine("Indexing one student......");
        var student = new Student { 
            Id = 100, 
            FirstName = "Paulo", 
            LastName = "Santos", 
            Gpa = 3.93, 
            GradYear = 2021 };v
        var response = client.Index<StringResponse>("students", "100",  
                                        PostData.Serializable(student));
        Console.WriteLine(response.Body);

        Console.WriteLine("Indexing many students......");
        var studentArray = new object[]
        {
            new { index = new { _index = "students", _type = "_doc", _id = "200"}},
            new {
                Id = 200, 
                FirstName = "Shirley", 
                LastName = "Rodriguez", 
                Gpa = 3.91, 
                GradYear = 2019},
            new { index = new { _index = "students", _type = "_doc", _id = "300"}},
            new {
                Id = 300, 
                FirstName = "Nikki", 
                LastName = "Wolf", 
                Gpa = 3.87, 
                GradYear = 2020}
        };

        var manyResponse = client.Bulk<StringResponse>(PostData.MultiJson(studentArray));

        Console.WriteLine(manyResponse.Body);


        Console.WriteLine("Searching for students who graduated in 2019......");
        var searchResponseLow = client.Search<StringResponse>("students",
            PostData.Serializable(
            new
            {
                from = 0,
                size = 20,

                query = new
                {
                    term = new
                    {
                        gradYear = new
                        {
                            value = 2019
                        }
                    }
                }
            }));

        Console.WriteLine(searchResponseLow.Body);

        Console.WriteLine("Searching for a student with the last name Santos......");

        var searchResponse = client.Search<StringResponse>("students",
            @" {
            ""query"":
                {
                    ""match"":
                    {
                        ""lastName"":
                        {
                            ""query"": ""Santos""
                        }
                    }
                }
            }");

        Console.WriteLine(searchResponse.Body);
    }
}
```
{% include copy.html %}


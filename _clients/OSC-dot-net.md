---
layout: default
title: 开始使用高级的.net客户端
nav_order: 10
has_children: false
parent: .NET客户端
---

# 开始使用高级的.net客户端 (OpenSearch.Client)

openSearch.client是高的-级别.NET客户端。它提供了强烈键入的请求和响应以及查询DSL。它可以通过提供模型来自动解析和序列化/序列化请求和响应，从而使您无法构建RAW JSON请求和解析RAW JSON响应。openSearch.client还公开了opensearch.net low-如果需要，请级别客户端。有关客户的完整API文档，请参阅[OpenSearch.Client API文档](https://opensearch-project.github.io/opensearch-net/api/OpenSearch.Client.html)。


该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-净仓库](https://github.com/opensearch-project/opensearch-net)。

## 安装OpenSearch.Client

要安装OpenSearch.Client，请下载[OpenSearch.Client Nuget软件包](https://www.nuget.org/packages/OpenSearch.Client/) 并将其添加到您的项目中。在Microsoft Visual Studio中，请执行以下步骤：
- 在里面**解决方案资源管理器** 面板，对-单击您的解决方案或项目并选择**管理Nuget软件包用于解决方案**。
- 搜索openSearch.client nuget软件包，然后选择**安装**。

另外，您可以将OpenSearch.Client添加到.csproj文件：
```xml
<Project>
  ...
  <ItemGroup>
    <PackageReference Include="OpenSearch.Client" Version="1.0.0" />
  </ItemGroup>
</Project>
```
{% include copy.html %}

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

默认情况下，OpenSearch.Client使用Camel Case将属性名称转换为字段名称。
{:.note}

## 连接到OpenSearch

创建OpenSearchClient对象时，请使用默认构造函数连接到默认OpenSearch host（`http://localhost:9200`）。

```cs
var client  = new OpenSearchClient();
```
{% include copy.html %}

要通过带有已知地址的单个节点连接到OpenSearch cluster，请在创建openSearch.client的实例时指定此地址：

```cs
var nodeAddress = new Uri("http://myserver:9200");
var client = new OpenSearchClient(nodeAddress);
```
{% include copy.html %}

您也可以通过多个节点连接到OpenSearch。使用节点池连接到OpenSearch集群，提供了诸如负载平衡和集群故障转移支持之类的优点。要使用多个节点连接到OpenSearch集群，请指定其地址并创建一个`ConnectionSettings` openSearch.client实例的对象：

```cs
var nodes = new Uri[]
{
    new Uri("http://myserver1:9200"),
    new Uri("http://myserver2:9200"),
    new Uri("http://myserver3:9200")
};

var pool = new StaticConnectionPool(nodes);
var settings = new ConnectionSettings(pool);
var client = new OpenSearchClient(settings);
```
{% include copy.html %}

## 使用Connectionsettings

`ConnectionConfiguration` 用于将配置选项传递给低点-Level OpenSearch.net客户端。`ConnectionSettings` 继承`ConnectionConfiguration` 并提供其他配置选项。
要设置没有指定索引名称的请求的节点的地址和默认索引名称，请创建一个`ConnectionSettings` 目的：

```cs
var node = new Uri("http://myserver:9200");
var config = new ConnectionSettings(node).DefaultIndex("students");
var client = new OpenSearchClient(config);
```
{% include copy.html %}

## 索引一个文档

创建一个学生实例：

```cs
var student = new Student { Id = 100, FirstName = "Paulo", LastName = "Santos", Gpa = 3.93, GradYear = 2021 };
```
{% include copy.html %}

要索引一个文档，您可以使用Fluent Lambda语法或对象初始化器语法。

将这个学生索引到`students` 使用Fluent Lambda语法的索引：

```cs
var response = client.Index(student, i => i.Index("students"));
```
{% include copy.html %}

将这个学生索引到`students` 使用对象初始化器语法的索引：

```cs
var response = client.Index(new IndexRequest<Student>(student, "students"));
```
{% include copy.html %}

## 索引许多文件

您可以使用opensearch.client的同时从集合中索引许多文档`IndexMany` 方法：

```cs
var studentArray = new Student[]
{
    new() {Id = 200, FirstName = "Shirley", LastName = "Rodriguez", Gpa = 3.91, GradYear = 2019},
    new() {Id = 300, FirstName = "Nikki", LastName = "Wolf", Gpa = 3.87, GradYear = 2020}
};

var manyResponse = client.IndexMany(studentArray, "students");
```
{% include copy.html %}

## 搜索文档

要搜索上面的学生，您想构建一个类似于以下查询DSL查询的查询：

```json
GET students/_search
{
  "query" : {
    "match": {
      "lastName": "Santos"
    }
  }
}
```

上面的查询是以下显式查询的速记版：

```json
GET students/_search
{
  "query" : {
    "match": {
      "lastName": {
        "query": "Santos"
      }
    }
  }
}
```

在opensearch.client中，此查询看起来像这样：

```cs
var searchResponse = client.Search<Student>(s => s
                                .Index("students")
                                .Query(q => q
                                    .Match(m => m
                                        .Field(fld => fld.LastName)
                                        .Query("Santos"))));
```
{% include copy.html %}

您可以通过在响应中访问文档来打印结果：

```cs
if (searchResponse.IsValid)
{
    foreach (var s in searchResponse.Documents)
    {
        Console.WriteLine($"{s.Id} {s.LastName} {s.FirstName} {s.Gpa} {s.GradYear}");
    }
}
```
{% include copy.html %}

响应包含一个文档，与正确的学生相对应：

`100 Santos Paulo 3.93 2021`

## 使用openSearch.client方法异步

对于需要异步代码的应用程序，OpenSearch中的所有方法调用都具有异步对应物：

```cs
// synchronous method
var response = client.Index(student, i => i.Index("students"));

// asynchronous method
var response = await client.IndexAsync(student, i => i.Index("students"));
```

## 倒下-Level OpenSearch.net客户端

openSearch.client暴露了低-将opensearch.net客户端升级，如果丢失了任何东西，可以使用：

```cs
var lowLevelClient = client.LowLevel;

var searchResponseLow = lowLevelClient.Search<SearchResponse<Student>>("students",
    PostData.Serializable(
        new
        {
            query = new
            {
                match = new
                {
                    lastName = new
                    {
                        query = "Santos"
                    }
                }
            }
        }));

if (searchResponseLow.IsValid)
{
    foreach (var s in searchResponseLow.Documents)
    {
        Console.WriteLine($"{s.Id} {s.LastName} {s.FirstName} {s.Gpa} {s.GradYear}");
    }
}
```
{% include copy.html %}

## 样本程序

以下是一个完整的示例程序，该程序说明了上述所有概念。它使用上面定义的学生课程。

```cs
using OpenSearch.Client;
using OpenSearch.Net;

namespace NetClientProgram;

internal class Program
{
    private static IOpenSearchClient osClient = new OpenSearchClient();

    public static void Main(string[] args)
    {       
        Console.WriteLine("Indexing one student......");
        var student = new Student { Id = 100, 
                                    FirstName = "Paulo", 
                                    LastName = "Santos", 
                                    Gpa = 3.93, 
                                    GradYear = 2021 };
        var response =  osClient.Index(student, i => i.Index("students"));
        Console.WriteLine(response.IsValid ? "Response received" : "Error");

        Console.WriteLine("Searching for one student......");
        SearchForOneStudent();

        Console.WriteLine("Searching using low-level client......");
        SearchLowLevel();

        Console.WriteLine("Indexing an array of Student objects......");
        var studentArray = new Student[]
        {
            new() { Id = 200, 
                    FirstName = "Shirley", 
                    LastName = "Rodriguez", 
                    Gpa = 3.91, 
                    GradYear = 2019},
            new() { Id = 300, 
                    FirstName = "Nikki", 
                    LastName = "Wolf", 
                    Gpa = 3.87, 
                    GradYear = 2020}
        };
        var manyResponse = osClient.IndexMany(studentArray, "students");
        Console.WriteLine(manyResponse.IsValid ? "Response received" : "Error");
    }

    private static void SearchForOneStudent()
    {
        var searchResponse = osClient.Search<Student>(s => s
                                .Index("students")
                                .Query(q => q
                                    .Match(m => m
                                        .Field(fld => fld.LastName)
                                        .Query("Santos"))));

        PrintResponse(searchResponse);
    }

    private static void SearchForAllStudentsWithANonEmptyLastName()
    {
        var searchResponse = osClient.Search<Student>(s => s
                                .Index("students")
                                .Query(q => q
                        						.Bool(b => b
                        							.Must(m => m.Exists(fld => fld.LastName))
                        							.MustNot(m => m.Term(t => t.Verbatim().Field(fld => fld.LastName).Value(string.Empty)))
                        						)));

        PrintResponse(searchResponse);
    }

    private static void SearchLowLevel()
    {
        // Search for the student using the low-level client
        var lowLevelClient = osClient.LowLevel;

        var searchResponseLow = lowLevelClient.Search<SearchResponse<Student>>
            ("students",
            PostData.Serializable(
                new
                {
                    query = new
                    {
                        match = new
                        {
                            lastName = new
                            {
                                query = "Santos"
                            }
                        }
                    }
                }));

        PrintResponse(searchResponseLow);
    }

    private static void PrintResponse(ISearchResponse<Student> response)
    {
        if (response.IsValid)
        {
            foreach (var s in response.Documents)
            {
                Console.WriteLine($"{s.Id} {s.LastName} " +
                    $"{s.FirstName} {s.Gpa} {s.GradYear}");
            }
        }
        else
        {
            Console.WriteLine("Student not found.");
        }
    }
}
```
{% include copy.html %}


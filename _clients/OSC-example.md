---
layout: default
title: 高级 .NET 客户端的更多高级功能
nav_order: 12
has_children: false
parent: .NET客户端
---

# 高级 .NET 客户端的更多高级功能 (OpenSearch.Client)

以下示例说明了opensearch.client的更高级功能。对于一个简单的示例，请参阅[入门指南]({{site.url}}{{site.baseurl}}/clients/OSC-dot-net/)。此示例使用以下学生课程。

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

## 映射

OpenSearch使用动态映射来推断索引的文档的字段类型。但是，要对文档的架构进行更多控制，您可以将明确的映射传递给OpenSearch。您可以在此映射中为文档的某些字段定义数据类型。

同样，OpenSearch.Client使用自动映射根据类的属性类型推断字段数据类型。要使用自动映射，请创建一个`students` 使用Automap的默认构造函数的索引：

```cs
var createResponse = await osClient.Indices.CreateAsync("students",
    c => c.Map(m => m.AutoMap<Student>()));
```
{% include copy.html %}

如果您使用自动映射，则将ID和Gradyear映射为整数，GPA被映射为双重映射，而FirstName和LastSname则作为带有关键字子字段的文本映射为文本。如果您想搜索firstName和lastname，并且仅允许案例-敏感的完整匹配，您可以通过将这些字段映射为关键字来抑制分析。在查询DSL中，您可以使用以下查询来完成此操作：

```json
PUT students
{
  "mappings" : {
    "properties" : {
      "firstName" : {
        "type" : "keyword"
      },
      "lastName" : {
        "type" : "keyword"
      }
    }
  }
}
```

在OpenSearch.Client中，您可以使用流体lambda语法将这些字段标记为关键字：

```cs
var createResponse = await osClient.Indices.CreateAsync(index,
                c => c.Map(m => m.AutoMap<Student>()
                .Properties<Student>(p => p
                .Keyword(k => k.Name(f => f.FirstName))
                .Keyword(k => k.Name(f => f.LastName)))));
```
{% include copy.html %}

## 设置

除映射外，您还可以在创建索引时指定设置，例如主和副本碎片的数量。以下查询将主要碎片的数量设置为1，将复制碎片的数量设置为2：

```json
PUT students
{
  "mappings" : {
    "properties" : {
      "firstName" : {
        "type" : "keyword"
      },
      "lastName" : {
        "type" : "keyword"
      }
    }
  }, 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2
  }
}
```

在openSearch.client中，相当于上述查询的等效内容如下：

```cs
var createResponse = await osClient.Indices.CreateAsync(index,
                            c => c.Map(m => m.AutoMap<Student>()
                            .Properties<Student>(p => p
                            .Keyword(k => k.Name(f => f.FirstName))
                            .Keyword(k => k.Name(f => f.LastName))))
                            .Settings(s => s.NumberOfShards(1).NumberOfReplicas(2)));
```
{% include copy.html %}

## 使用批量API索引多个文档

除了使用一个文档索引`Index` 和`IndexDocument` 并使用多个文档索引`IndexMany`，您可以通过使用对文档索引的更多控制`Bulk` 或者`BulkAll`。索引文档单独效率低下，因为它会为发送的每个文档创建HTTP请求。Bulkall Helper使您无法处理重试，分块或退缩请求功能。如果请求发生故障，它将自动重新检索，如果服务器下降，请备份，并控制一个HTTP请求中发送了多少个文档。

在以下示例中，`BulkAll` 配置有索引名称，返回次数的数量以及返回时间。此外，并行性设置的最大程度控制包含数据的并行HTTP请求的数量。最后，大小参数信号在一个HTTP请求中发送了多少个文档。

我们建议将大小设置为生产中的100–1000个文档。
{: .tip}

`BulkAll` 获取数据流并返回可观察到的可观察到背景操作的可观察到的。

```cs
var bulkAll = osClient.BulkAll(ReadData(), r => r
            .Index(index)
            .BackOffRetries(2)
            .BackOffTime("30s")
            .MaxDegreeOfParallelism(4)
            .Size(100));
```
{% include copy.html %}

## 与布尔查询一起搜索

OpenSearch.Client公开完整的OpenSearch查询功能。除了使用匹配查询的简单搜索外，您还可以创建一个更复杂的布尔查询，以搜索2022年毕业并按姓氏对其进行排序的学生。在下面的示例中，搜索仅限于10个文档，并且滚动API用于控制结果的分页。

```cs
var gradResponse = await osClient.SearchAsync<Student>(s => s
                        .Index(index)
                        .From(0)
                        .Size(10)
                        .Scroll("1m")
                        .Query(q => q
                        .Bool(b => b
                        .Filter(f => f
                        .Term(t => t.Field(fld => fld.GradYear).Value(2022)))))
                        .Sort(srt => srt.Ascending(f => f.LastName)));
```
{% include copy.html %}

响应包含文档属性，其中包含来自OpenSearch的匹配文档。这些数据是学生类型的必不可少的JSON对象的形式，因此您可以以强烈的打字方式访问其属性。所有序列化和避难所都由OpenSearch.Client处理。

## 聚合

OpenSearch.Client包括完整的OpenSearch查询功能，包括聚合。除了将搜索结果分组到水桶中（例如，按GPA范围对学生进行分组），您还可以计算诸如总和或平均值之类的指标。以下查询计算了索引中所有学生的平均GPA。

将大小设置为0表示OpenSearch只能返回聚合，而不是实际文档。
{: .tip}

```cs
var aggResponse = await osClient.SearchAsync<Student>(s => s
                                .Index(index)
                                .Size(0)
                                .Aggregations(a => a
                                .Average("average gpa", 
                                            avg => avg.Field(fld => fld.Gpa))));
```
{% include copy.html %}

## 用于创建索引和索引数据的示例程序

以下程序创建索引，读取逗号的学生记录流-将文件分开，并将这些数据索引到OpenSearch中。

```cs
using OpenSearch.Client;

namespace NetClientProgram;

internal class Program
{
    private const string index = "students";

    public static IOpenSearchClient osClient = new OpenSearchClient();

    public static async Task Main(string[] args)
    {
        // Check if the index with the name "students" exists
        var existResponse = await osClient.Indices.ExistsAsync(index);

        if (!existResponse.Exists)  // There is no index with this name
        {
            // Create an index "students"
            // Map FirstName and LastName as keyword
            var createResponse = await osClient.Indices.CreateAsync(index,
                c => c.Map(m => m.AutoMap<Student>()
                .Properties<Student>(p => p
                .Keyword(k => k.Name(f => f.FirstName)) 
                .Keyword(k => k.Name(f => f.LastName))))
                .Settings(s => s.NumberOfShards(1).NumberOfReplicas(1)));

            if (!createResponse.IsValid && !createResponse.Acknowledged)
            {
                throw new Exception("Create response is invalid.");
            }

            // Take a stream of data and send it to OpenSearch
            var bulkAll = osClient.BulkAll(ReadData(), r => r
            .Index(index)
            .BackOffRetries(2)
            .BackOffTime("20s")
            .MaxDegreeOfParallelism(4)
            .Size(10));

            // Wait until the data upload is complete.
            // FromMinutes specifies a timeout.
            // r is a response object that is returned as the data is indexed.
            bulkAll.Wait(TimeSpan.FromMinutes(10), r => 
                Console.WriteLine("Data chunk indexed"));
        }
    }

    // Reads student data in the form "Id,FirsName,LastName,GradYear,Gpa"
    public static IEnumerable<Student> ReadData()
    {
        var file = new StreamReader("C:\\search\\students.csv");

        string s;
        while ((s = file.ReadLine()) is not null)
        {
            yield return new Student(s);
        }
    }
}
```
{% include copy.html %}

## 搜索样本程序

以下程序按名称和毕业日期搜索学生，并计算平均GPA。

```cs
using OpenSearch.Client;

namespace NetClientProgram;

internal class Program
{
    private const string index = "students";

    public static IOpenSearchClient osClient = new OpenSearchClient();

    public static async Task Main(string[] args)
    {
        await SearchByName();

        await SearchByGradDate();

        await CalculateAverageGpa();
    }

    private static async Task SearchByName()
    {
        Console.WriteLine("Searching for name......");

        var nameResponse = await osClient.SearchAsync<Student>(s => s
                                .Index(index)
                                .Query(q => q
                                .Match(m => m
                                .Field(fld => fld.FirstName)
                                .Query("Zhang"))));

        if (!nameResponse.IsValid)
        {
            throw new Exception("Aggregation query response is not valid.");
        }

        foreach (var s in nameResponse.Documents)
        {
            Console.WriteLine($"{s.Id} {s.LastName} " +
                $"{s.FirstName} {s.Gpa} {s.GradYear}");
        }
    }

    private static async Task SearchByGradDate()
    {
        Console.WriteLine("Searching for grad date......");

        // Search for all students who graduated in 2022
        var gradResponse = await osClient.SearchAsync<Student>(s => s
                                .Index(index)
                                .From(0)
                                .Size(2)
                                .Scroll("1m")
                                .Query(q => q
                                .Bool(b => b
                                .Filter(f => f
                                .Term(t => t.Field(fld => fld.GradYear).Value(2022)))))
                                .Sort(srt => srt.Ascending(f => f.LastName))
                                .Size(10));


        if (!gradResponse.IsValid)
        {
            throw new Exception("Grad date query response is not valid.");
        }

        while (gradResponse.Documents.Any())
        {
            foreach (var data in gradResponse.Documents)
            {
                Console.WriteLine($"{data.Id} {data.LastName} {data.FirstName} " +
                    $"{data.Gpa} {data.GradYear}");
            }
            gradResponse = osClient.Scroll<Student>("1m", gradResponse.ScrollId);
        }
    }

    public static async Task CalculateAverageGpa()
    {
        Console.WriteLine("Calculating average GPA......");

        // Search and aggregate
        // Size 0 means documents are not returned, only aggregation is returned
        var aggResponse = await osClient.SearchAsync<Student>(s => s
                                .Index(index)
                                .Size(0)
                                .Aggregations(a => a
                                .Average("average gpa", 
                                            avg => avg.Field(fld => fld.Gpa))));
       
        if (!aggResponse.IsValid) throw new Exception("Aggregation response not valid");

        var avg = aggResponse.Aggregations.Average("average gpa").Value;
        Console.WriteLine($"Average GPA is {avg}");
    }
}
```
{％包括copy.html％


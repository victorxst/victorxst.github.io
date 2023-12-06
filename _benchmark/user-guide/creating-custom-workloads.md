---
layout: default
title: 创建自定义工作负载
nav_order: 10
parent: 用户指南
redirect_from: /benchmark/creating-custom-workloads/
---

# 创建自定义工作负载

OpenSearch基准包括一组[工作负载](https://github.com/opensearch-project/opensearch-benchmark-workloads) 您可以使用群集基准数据。此外，如果要创建针对自己数据量身定制的工作负载，则可以使用以下一个选项之一来创建自定义工作负载：

- [创建自定义工作负载](#creating-custom-workloads)
  - [从现有集群中创建工作量](#creating-a-workload-from-an-existing-cluster)
    - [先决条件](#prerequisites)
    - [自定义工作量](#customizing-the-workload)
    - [在没有现有集群的情况下创建工作量](#creating-a-workload-without-an-existing-cluster)
  - [调用您的自定义工作量](#invoking-your-custom-workload)
  - [高级选项](#advanced-options)
    - [测试模式](#test-mode)
    - [将差异添加到测试程序](#adding-variance-to-test-procedures)
    - [单独的操作和测试程序](#separate-operations-and-test-procedures)
  - [下一步](#next-steps)

## 从现有集群中创建工作量

如果您已经拥有带有索引数据的OpenSearch集群，请使用以下步骤为群集创建自定义工作负载。

### 先决条件

在创建自定义工作负载之前，请确保您有以下先决条件：

- 一个带有索引的OpenSearch集群，其中包含1000个或更多文档。如果您的集群的索引至少包含1000个文档，则工作负载仍然可以运行测试，但是，您无法使用工作负载运行`--test-mode`。
- 您必须拥有正确的权限才能访问OpenSearch集群。有关集群权限的更多信息，请参阅[权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/)。

### 自定义工作量

要开始创建自定义工作量，请使用`opensearch-benchmark create-workload` 命令。

```
opensearch-benchmark create-workload \
--workload="<WORKLOAD NAME>" \
--target-hosts="<CLUSTER ENDPOINT>" \
--client-options="basic_auth_user:'<USERNAME>',basic_auth_password:'<PASSWORD>'" \
--indices="<INDEXES TO GENERATE WORKLOAD FROM>" \
--output-path="<LOCAL DIRECTORY PATH TO STORE WORKLOAD>"
```

将以下示例中的以下选项替换为特定于您现有群集的信息：

- `--workload`：自定义工作负载的自定义名称。
- `--target-hosts:` 逗号-主机的分离列表：端口对从中提取数据。
- `--client-options`：OpenSearch基准测试用来访问群集的基本身份验证客户端选项。
- `--indices`：包含数据的OpenSearch集群中的一个或多个索引。
- `--output-path`：OpenSearch基准标准创建工作负载及其配置文件的目录。

以下示例响应创建了一个名为的工作负载`movies` 来自带有索引的群集`movies-info`。这`movies-info` 索引包含2,000多个文件。

```
   ____                  _____                      __       ____                  __                         __
  / __ \____  ___  ____ / ___/___  ____ ___________/ /_     / __ )___  ____  _____/ /_  ____ ___  ____ ______/ /__
 / / / / __ \/ _ \/ __ \\__ \/ _ \/ __ `/ ___/ ___/ __ \   / __  / _ \/ __ \/ ___/ __ \/ __ `__ \/ __ `/ ___/ //_/
/ /_/ / /_/ /  __/ / / /__/ /  __/ /_/ / /  / /__/ / / /  / /_/ /  __/ / / / /__/ / / / / / / / / /_/ / /  / ,<
\____/ .___/\___/_/ /_/____/\___/\__,_/_/   \___/_/ /_/  /_____/\___/_/ /_/\___/_/ /_/_/ /_/ /_/\__,_/_/  /_/|_|
    /_/

[INFO] You did not provide an explicit timeout in the client options. Assuming default of 10 seconds.
[INFO] Connected to OpenSearch cluster [380d8fd64dd85b5f77c0ad81b0799e1e] version [1.1.0].

Extracting documents for index [movies] for test mode...      1000/1000 docs [100.0% done]
Extracting documents for index [movies]...                    2000/2000 docs [100.0% done]

[INFO] Workload movies has been created. Run it with: opensearch-benchmark --workload-path=/Users/hoangia/Desktop/workloads/movies

-------------------------------
[INFO] SUCCESS (took 2 seconds)
-------------------------------
```

作为创建工作负载的一部分，OpenSearch基准标准生成以下文件。您可以在由`--output-path` 选项。

- `workload.json`：包含一般的工作量规格。
- `<index>.json`：包含提取索引的映射和设置。
- `<index>-documents.json`：包含提取索引中每个文档的来源。任何来源的后缀`-1k` 仅包括工作负载的文档语料库的一小部分，仅在测试模式下运行工作负载时才使用。

默认情况下，OpenSearch基准标准不包含引用生成查询的引用。因为您对数据有最好的理解，所以我们建议将查询添加到`workload.json` 这与您的索引规格相匹配。使用以下内容`match_all` 查询作为查询的示例添加到您的工作量中：

```json
{
      "operation": {
        "name": "query-match-all",
        "operation-type": "search",
        "body": {
          "query": {
            "match_all": {}
          }
        }
      },
      "clients": 8,
      "warmup-iterations": 1000,
      "iterations": 1000,
      "target-throughput": 100
    }
```

### 在没有现有集群的情况下创建工作量

如果要创建自定义工作负载，但没有带有索引数据的现有OpenSearch集群，则可以通过直接构建Workload源文件来创建工作负载。您需要的只是可以导出为JSON格式的数据。

要使用源文件构建工作负载，请为您的工作负载创建目录并执行以下步骤：

1. 建个`<index>-documents.json` 包含包含工作量文档文件的文档的行，并容纳要摄入并查询到集群中的所有数据。以下示例显示了`movies-documents.json` 包含有关著名电影的文档的文件：

   ```json
  # First few rows of movies-documents.json
  {"title": "Back to the Future", "director": "Robert Zemeckis", "revenue": "$212,259,762 USD", "rating": "8.5 out of 10",  "image_url": "https://imdb.com/images/32"}
  {"title": "Avengers: Endgame", "director": "Anthony and Joe Russo", "revenue": "$2,800,000,000 USD", "rating": "8.4 out   of 10", "image_url": "https://imdb.com/images/2"}
  {"title": "The Grand Budapest Hotel", "director": "Wes Anderson", "revenue": "$173,000,000 USD", "rating": "8.1 out of 10", "image_url": "https://imdb.com/images/65"}
  {"title": "The Godfather: Part II", "director": "Francis Ford Coppola", "revenue": "$48,000,000 USD", "rating": "9 out of 10", "image_url": "https://imdb.com/images/7"}
   ```

2. In the same directory, build a `index.json` file. The workload uses this file as a reference for data mappings and index settings for the documents contained in `<index>-Documents.json`. The following example creates mappings and settings specific to the `电影-Documents.json` data from the previous step:

    ```json
    {
    "settings": {
        "index.number_of_replicas": 0
    },
    "mappings": {
        "dynamic": "strict",
        "properties": {
        "title": {
            "type": "text"
        },
        "director": {
            "type": "text"
        },
        "revenue": {
            "type": "text"
        },
        "rating": {
            "type": "text"
        },
        "image_url": {
            "type": "text"
        }
        }
    }
    }
    ```

3. Next, build a `工作负载` file that provides a high-level overview of your workload and determines how your workload runs benchmark tests. The `工作负载` file contains the following sections:

   - `指数`: Defines the name of the index to be created in your OpenSearch cluster using the mappings from the workload's `index.json` 在上一步中创建的文件。
   - `corpora`：定义语料库和源文件，包括：
      - `document-count`：中的文档数量`<index>-documents.json`。要获取准确数量的文档，请运行`wc -l <index>-documents.json`。
      - `uncompressed-bytes`：索引内的字节数。要获得准确数量的字节，请运行`stat -f %z <index>-documents.json` 在macos或`stat -c %s <index>-documents.json` 在GNU/Linux上。或者，运行`ls -lrt | grep <index>-documents.json`。
   - `schedule`：定义工作负载的操作顺序和可用测试程序。

以下示例`workload.json` 文件提供了`movies` 工作量。这`indices` 部分创建一个称为的索引`movies`。Corpora部分是指在第一步中创建的源文件，`movie-documents.json`，并提供文档计数和未压缩字节的数量。最后，时间表部分定义了一些工作负载在调用时执行的操作，包括：

- 删除任何名称的当前索引`movies`。
- 创建一个名称的索引`movies` 基于数据`movie-documents.json` 和来自`index.json`。
- 验证集群身体健康，可以摄取新指数。
- 从中摄入数据语料库`workload.json` 进入集群。
- 查询结果。

    ```json
    {
    "version": 2,
    "description": "Tutorial benchmark for OpenSearch Benchmark",
    "indices": [
        {
        "name": "movies",
        "body": "index.json"
        }
    ],
    "corpora": [
        {
        "name": "movies",
        "documents": [
            {
            "source-file": "movies-documents.json",
            "document-count": 11658903, # Fetch document count from command line
            "uncompressed-bytes": 1544799789 # Fetch uncompressed bytes from command line
            }
        ]
        }
    ],
    "schedule": [
        {
        "operation": {
            "operation-type": "delete-index"
        }
        },
        {
        "operation": {
            "operation-type": "create-index"
        }
        },
        {
        "operation": {
            "operation-type": "cluster-health",
            "request-params": {
            "wait_for_status": "green"
            },
            "retry-until-success": true
        }
        },
        {
        "operation": {
            "operation-type": "bulk",
            "bulk-size": 5000
        },
        "warmup-time-period": 120,
        "clients": 8
        },
        {
        "operation": {
            "operation-type": "force-merge"
        }
        },
        {
        "operation": {
            "name": "query-match-all",
            "operation-type": "search",
            "body": {
            "query": {
                "match_all": {}
            }
            }
        },
        "clients": 8,
        "warmup-iterations": 1000,
        "iterations": 1000,
        "target-throughput": 100
        }
    ]
    }
    ```

The corpora section refers to the source file created in step one, `电影-Documents.json`，并提供文档计数和未压缩字节的数量。最后，时间表部分定义了一些工作负载在调用时执行的操作，包括：

- 删除任何名称的当前索引`movies`。
- 创建一个名称的索引`movies` 基于数据`movie-documents.json` 和来自`index.json`。
   - 验证集群身体健康，可以摄取新指数。
   - 从中摄入数据语料库`workload.json` 进入集群。
   - 查询结果。



对于所有创建的工作负载文件，请通过运行测试来验证工作负载是否正常。要验证工作负载，请运行以下命令，更换`--workload-path` 有了通往工作负载目录的途径：

```
opensearch-benchmark list workloads --workload-path=</path/to/workload/>
```

## 调用您的自定义工作量

使用`opensearch-benchmark execute-test` 命令调用您的新工作负载并针对OpenSearch集群运行基准测试，如下示例所示。代替`--workload-path` 有了通往自定义工作量的路径，`--target-host` 与`host:port` 群集的成对，`--client-options` 访问集群所需的任何授权选项。

```
opensearch-benchmark execute_test \
--pipeline="benchmark-only" \
--workload-path="<PATH OUTPUTTED IN THE OUTPUT OF THE CREATE-WORKLOAD COMMAND>" \
--target-host="<CLUSTER ENDPOINT>" \
--client-options="basic_auth_user:'<USERNAME>',basic_auth_password:'<PASSWORD>'"
```

测试的结果出现在设置的目录中`--output-path` 选项in`workloads.json`。

## 高级选项

您可以通过以下高级选项来增强自定义工作负载的功能。

### 测试模式

如果要以测试模式进行测试以确保您的工作负载按预期运行，请添加`--test-mode` 选项`execute-test` 命令。测试模式仅摄入每个索引中的第一个1000个文档，并针对它们运行查询操作。

要使用测试模式，请创建一个`<index>-documents-1k.json` 包含来自第一个1000个文档的文件`<index>-documents.json` 使用以下命令：

```
head -n 1000 <index>-documents.json > <index>-documents-1k.json
```

然后，运行`opensearch-benchmark execute-test` 与选项`--test-mode`。测试模式运行工作负载测试的快速版本。

```
opensearch-benchmark execute_test \
--pipeline="benchmark-only"  \
--workload-path="<PATH OUTPUTTED IN THE OUTPUT OF THE CREATE-WORKLOAD COMMAND>" \
--target-host="<CLUSTER ENDPOINT>" \
--client-options"basic_auth_user:'<USERNAME>',basic_auth_password:'<PASSWORD>'" \
--test-mode
```

### 将差异添加到测试程序

使用自定义工作负载几次后，您可能需要使用相同的工作负载，但要以不同的顺序执行工作负载的操作。您可以提供测试程序来改变工作负载操作，而不是直接创建新的工作量或重组过程。

要增加工作量操作的差异，请转到您的`workload.json` 文件并替换`schedule` 部分`test_procedures` 数组，如下面的示例所示。数组中的每个项目都包含以下内容：

- `name`：测试过程的名称。
- `default`：设置为`true`，OpenSearch基准默认为指定的测试过程`default` 在工作量中，如果未指定其他测试程序。
- `schedule`：所有操作都将运行测试过程。


```json
"test_procedures": [
    {
      "name": "index-and-query",
      "default": true,
      "schedule": [
        {
          "operation": {
            "operation-type": "delete-index"
          }
        },
        {
          "operation": {
            "operation-type": "create-index"
          }
        },
        {
          "operation": {
            "operation-type": "cluster-health",
            "request-params": {
              "wait_for_status": "green"
            },
            "retry-until-success": true
          }
        },
        {
          "operation": {
            "operation-type": "bulk",
            "bulk-size": 5000
          },
          "warmup-time-period": 120,
          "clients": 8
        },
        {
          "operation": {
            "operation-type": "force-merge"
          }
        },
        {
          "operation": {
            "name": "query-match-all",
            "operation-type": "search",
            "body": {
              "query": {
                "match_all": {}
              }
            }
          },
          "clients": 8,
          "warmup-iterations": 1000,
          "iterations": 1000,
          "target-throughput": 100
        }
      ]
    }
  ]
}
```

### 单独的操作和测试程序

如果你想让你的`workload.json` 文件更可读，您可以将操作和测试过程分为不同目录，并将每个目录的路径引入到`workload.json`。要分开操作和过程，请执行以下步骤：

1. 将所有测试过程添加到一个文件中。您可以给文件提供任何名称。因为`movies` 前面的工作负载包含和索引任务和查询，此步骤名称测试过程文件`index-and-query.json`。
2. 将所有操作添加到名称的文件中`operations.json`。
3. 在`workloads.json` 通过添加以下语法，替换`parts` 如以下示例所示，具有每个文件的相对路径：

    ```json
    "operations": [
        {% raw %}{{ benchmark.collect(parts="operations/*.json") }}{% endraw %}
    ]
    # Reference test procedure files in workload.json
    "test_procedures": [
        {% raw %}{{ benchmark.collect(parts="test_procedures/*.json") }}{% endraw %}
    ]
    ```

## 下一步

- 有关配置OpenSearch基准测试的更多信息，请参阅[配置OpenSearch基准测试]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark/)。
- 要显示针对OpenSearch基准的预包装工作负载列表，请参见[OpenSearch-基准-工作负载](https://github.com/opensearch-project/opensearch-benchmark-workloads) 存储库。


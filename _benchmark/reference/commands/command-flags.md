---
layout: default
title: 命令标志
nav_order: 51
parent: 命令参考
redirect_from: /benchmark/commands/command-flags/
grand_parent: OpenSearch基准参考
---

# 命令标志
OpenSearch基准测试使用命令行标志来改变基准的行为。并非所有标志都可以与每个命令一起使用。要找出特定命令支持哪些标志，请输入`opensearch-benchmark <command> --h`。

使用以下语法将所有命令标志添加到命令中：

```bash
opensearch-benchmark <command> --<command-flag>
```

接受逗号的国旗-分开的值`--telemetry`，也可以接受JSON数组。可以通过传递文件路径结束的文件路径来定义`.json` 或作为JSON字符串内联。

- 逗号-分离值：`opensearch-benchmark ... --test-procedure="ingest-only,search-aggregations"`
- JSON文件：`opensearch-benchmark ... --workload-params="params.json"`
- JSON内联字符串：`opensearch-benchmark  ... --telemetry='["node-stats", "recovery-stats"]'`

## 工作量-小路

这可以是包含一个的目录`workload.json` 文件或a`.json` 具有包含工作负载规范的任意名称的文件。`--workload-path` 和`--workload-repository` 也`--workload` 是互斥的。

## 工作量-存储库

这定义了OpenSearch基准加载工作负载的存储库。`--workload-path` 和`--workload-repository` 也`--workload` 是互斥的。

## 工作量-修订

定义了OpenSearch基准测试应使用的工作负载源树的特定修订。

## 工作量

根据工作负载的名称定义要使用的工作负载。您可以使用`opensearch-benchmark list workloads`。`--workload-path` 和`--workload-repository` 也`--workload` 是互斥的。

## 工作量-参数
定义哪些变量要注入工作负载。注入的变量必须在工作负载中可用。要查看官方工作负载中哪些参数有效，请从[工作负载存储库](https://github.com/opensearch-project/opensearch-benchmark-workloads)。

## 测试-程序

定义使用的测试程序。您可以使用`opensearch-benchmark list test-procedures`。

## 测试-执行-ID

为测试运行定义唯一的ID。

## 包括-任务

定义一个逗号-分开运行的测试过程列表。默认情况下，运行测试过程中列出的所有任务均已运行。

测试按定义的顺序执行`test-procedure`---并非按照命令中定义它们的顺序。

所有任务过滤器均敏感。

## 排除-任务

定义一个逗号-分开的测试过程任务列表不运行。

## 基线

用于比较竞争者testexecution的基线测试驱动ID。

## 竞争者

比较竞争者的测试表ID与基线相比。

## 结果-格式

定义命令行结果的输出格式`markdown` 或者`csv`。默认为`markdown`。

## 结果-数字-对齐

定义列的列号对齐`compare` 命令输出结果。默认为`right`。

## 结果-文件

当提供文件路径时，将结果与路径中指示的文件进行比较。

## 展示-在-结果

确定是否在结果文件中包括比较。

## 条款-config-存储库

定义opensearch基准加载的存储库`provision-configs` 和`provision-config-instances`。

## 条款-config-修订

定义特定的git修订`provision-config` 该OpenSearch基准应使用。

## 条款-config-小路

定义通往`--provision-config-instance` 以及要使用的任何OpenSearch插件配置。

## 分配-版本

根据版本号下载指定的OpenSearch发行版。有关已发布的OpenSearch版本列表，请参见[版本历史记录](https://opensearch.org/docs/version-history/)。

## 分配-存储库

定义应从中下载OpenSearch发行版的存储库。默认为`release`。

## 条款-config-实例

定义`--provision-config-instance` 使用。您可以使用命令查看可能的配置实例`opensearch-benchmark list provision-config-instances`。

## 条款-config-实例-参数

逗号-钥匙的分开列表-价值对逐字注射为变量`provision-config-instance`。

## 目标-主持人

定义一个逗号-分开的主机列表-如果使用管道，则应针对的端口对`benchmark-only`。默认为`localhost:9200`。

## 目标-操作系统

应下载OpenSearch工件的目标操作系统（OS）。默认值是当前操作系统。

## 目标-拱

应下载工件的CPU体系结构的名称。

## 修订

定义当前的源代码修订，用于运行基准测试。默认为`current`。

此命令标志可以使用以下选项：

   - `current`：根据您的OpenSearch发行版，使用源树的当前修订版。
   - `latest`：从源树的主要分支中获取最新的修订。

您也可以使用时间戳或从源树提交ID。使用时间戳时，指定`@ts`， 在哪里"ts" 是有效的ISO 8601时间戳，例如`@2013-07-27T10:37:00Z`。

## OpenSearch-插件

定义哪个[OpenSearch插件]({{site.url}}{{site.baseurl}}/install-and-configure/plugins/) 安装。默认情况下，没有安装插件。

## 插入-参数

定义一个逗号-钥匙的分开列表-逐字注入所有插件的值对作为变量。

## 运行-JDK

JDK使用的主要版本。

## 客户-选项

定义一个逗号-分开使用的客户列表。所有选项都传递给OpenSearch Python客户端。默认为`timeout:60`。

## 加载-工人-协调员-主持人

定义一个逗号-分开的主机列表，这些主机坐标负载。默认为`localhost`。

## 使能够-工人-协调员-分析

启用OpenSearch基准测试工人协调员的性能分析。默认为`false`。

## 管道

这`--pipeline` 选项选择要运行的管道。您可以通过运行来找到由OpenSearch基准支持的管道列表`opensearch-benchmark list pipelines`。

## 遥测

当使用逗号提供设备时，启用提供的遥测设备-分开列表。您可以使用`opensearch-benchmark list telemetry`。

## 遥测-参数

启用为遥测设备设置参数。接受逗号列表-分开的钥匙-值对，每个对由结肠或JSON文件名界定。

## 在-错误

控制OpenSearch基准测试如何响应错误。默认为`continue`。

您可以使用此命令标志使用以下选项：

- `continue`：尽管出错，仍继续进行测试。
- `abort`：在发生错误时中止测试。

## 保存-安装

保持基准候选人及其指数。默认为`false`。

## 杀-跑步-过程

设置为`true`，停止当前正在运行的任何OpenSearch基准过程，并允许基准测试继续运行。默认为`false`。


## 图表-规格-小路

将路径设置为包含可用于生成图表的图表规范的JSON文件。

## 图表-类型

生成指示的图表类型`time-series` 或者`bar`。默认为`time-series`。

## 输出-小路

图表输出使用的名称和路径。默认为`stdout`。

## 限制

限制最近测试运行的搜索结果数。默认为`10`


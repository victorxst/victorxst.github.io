---
layout: default
title: 来源协调
nav_order: 35
parent: 管理数据预先
---

# 来源协调

_ source coordination_是在数据源之间进行协调和分发工作的概念-节点环境。一些数据源，例如亚马逊运动量或亚马逊简单队列服务（Amazon SQS），在本地处理协调。其他数据源，例如OpenSearch，Amazon简单存储服务（Amazon S3），Amazon DynamoDB和JDBC/ODBC，不支持源协调。

数据PEPPER源协调决定了Data Prepper群集中每个节点执行的工作分区，并防止重复的工作分区。

受到启发[运动端客户库](https://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html)，Data Prepper以租赁形式利用分布式商店来处理工作的分布和重复数据删除。

## 格式分区

源协调将资源分为"partitions of work." 例如，S3对象将是Amazon S3的工作分区，或者OpenSearch索引将是OpenSearch的工作分区。

Data Prepper采用源选择的每个工作分区，并在分布式商店中创建相应的项目，该项目Prepper使用用于源协调。这些项目中的每一个都有以下标准格式，可以通过分布式商店实施来扩展。

| 价值| 类型| 描述|
| :--- | :--- | :--- |
| `sourceIdentifier` | 细绳| 数据Prepper管道在此分区上起作用的标识符。默认情况下，`sourceIdentifier` 由子前缀-管道名称，但是可以配置附加的前缀`partition_prefix` 在您的数据中-预先-config.yaml文件。|
| `sourcePartitionKey` | 细绳| 与此项目相关的工作分区的标识符。例如，对于`s3` 具有扫描功能的来源，此标识符是S3桶的`objectKey` 组合。
| `partitionOwner` | 细绳| 积极拥有并正在研究此分区的节点的标识符。此ID包含节点的主机名，但`null` 当该分区不拥有时。|
| `partitionProgressState` | 细绳| 在恢复另一个节点的情况下，源源可能需要的json字符串对象，该对象代表了工作的分区或源可能需要的任何其他元数据，而在崩溃期间最后一个节点停止了。|
| `partitionOwnershipTimeout` | 时间戳| 每当数据预先节点获取分区时，10-将分钟的超时时间发送给分区的所有者，以处理节点崩溃的事件。当所有者保存分区状态时，所有权将续签另外10分钟。|
| `sourcePartitionStatus` | 枚举| 代表分区的当前状态：`ASSIGNED` 意味着该分区目前正在处理，`UNASSIGNED` 意味着分区正在等待处理，`CLOSED` 意味着该分区正在等待以后处理，并且`COMPLETED` 意味着该分区已经被处理。|           
| `reOpenAt` | 时间戳| 表示重新打开封闭分区并被认为可用于处理的时间。仅适用于封闭分区。|
| `closedCount` | 长的| 跟踪该分区被标记为多少次`CLOSED`。|


## 获取分区

分区是按顺序获取的，即将它们退还在`List<PartitionIdentifer>` 由来源提供。当节点试图获取分区时，数据预珀执行以下步骤：

1. 数据预先查询`ASSIGNED` 分区检查是否有`ASSIGNED` 分区已过期分区所有者。这旨在将在处理中间有节点崩溃的分区分配优先级，这可以允许使用可能对时间敏感的分区状态。
2. 查询后`ASSIGNED` 分区，数据预先查询`CLOSED` 分区以确定是否有任何分区`reOpenAt` 时间戳已达到。
3. 如果没有`ASSIGNED` 或者`CLOSED` 可用分区，然后数据预先查询`UNASSIGNED` 分区直到这些分区为止`ASSIGNED`。

如果该流程发生并且该节点没有获取任何分区，则分区供应商功能`getNextPartition` 的方法`SourceCoordinator` 将创建新的分区。供应商功能完成后，数据预备再次查询分区的`ASSIGNED`，`CLOSED`， 和`UNASSIGNED`。

## 全球国家

传递给`getNextPartition` 方法以全球状态创建新的分区`Map<String, Object>`。该状态在群集中的所有节点之间共享，并且仅由一个源确定的一次由一个节点运行。

## 配置

下表提供了可选的配置值`source_coordination`。

| 价值| 类型| 描述|
| :--- | :--- | :--- |
| `partition_prefix` | 细绳| 一个前缀`sourceIdentifier` 用于区分共享相同分布式商店的数据预先群集。|
| `store` | 目的| 包含用于使用商店的配置的对象，键是商店的名称，例如`in_memory` 或者`dynamodb`，该值是该商店类型上可用的任何配置。|

### 支持的商店
从数据prepper 2.4开始，仅`in_memory` 和`dynamodb` 支持商店：

- 这`in_memory` 商店是
默认值时`source_coordination` 设置在`data-prepper-config.yaml` 文件，仅应用于单个-节点配置。
- 这`dynamodb` 商店用于多种-节点数据预先环境。这`dynamodb` 可以在需要利用源协调的一个或多个数据预先群集之间共享存储。

#### DynamoDB商店

数据Prepper将尝试创建`dynamodb` 启动表，除非`skip_table_creation` 标志已配置为`true`。可选，您可以配置[时间-到-居住](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) （（`ttl`）在桌子上，这导致商店随着时间的推移清理项目。有些来源依靠源协调来重复数据删除，因此请务必配置足够大的数据`ttl` 对于管道持续时间。

如果`ttl` 未在表上配置，表格中不再需要任何项目。

以下显示了Data Prepper创建表所需的完整权限，请启用`ttl`，并与表相互作用：

```json
{
  "Sid": "ReadWriteSourceCoordinationDynamoStore",
  "Effect": "Allow",
  "Action": [
    "dynamodb:DescribeTimeToLive",
    "dynamodb:UpdateTimeToLive",
    "dynamodb:DescribeTable",
    "dynamodb:CreateTable",
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query"
  ],
  "Resource": [
    "arn:aws:dynamodb:${REGION}:${AWS_ACCOUNT_ID}:table/${TABLE_NAME}",
    "arn:aws:dynamodb:${REGION}:${AWS_ACCOUNT_ID}:table/${TABLE_NAME}/index/source-status"
  ]
}
```


| 价值| 必需的| 类型| 描述|
| :--- | :--- | :--- | :--- | 
| `table_name` | 是的| 细绳| 用于源协调的表名称。|
| `region` | 是的| 细绳| DynamoDB表的区域。|
| `sts_role_arn` | 不| 细绳|  这`sts` 包含表权限的角色。如果不提供时，请使用默认凭据。|
| `sts_external_id` | 不| 细绳| API调用中使用的外部ID假设`sts_role_arn`。|
| `skip_table_creation` | 不| 布尔| 如果设置为`true` 当使用现有商店时，尝试创建商店的尝试将跳过。默认为`false`。|
| `provisioned_write_capacity_units` | 不| 整数|  在表上配置的写入容量单位的数量。默认为`10`。|
| `provisioned_read_capacity_units`  | 不| 整数| 读取容量单元的数量要在表上配置。默认为`10`。|
| `ttl` | 期间| 选修的。表中项目的TTL持续时间。当对项目进行更新时，TTL会随着此持续时间扩展。默认值在表上不使用TTL。|
  
以下示例显示了`dynamodb` 店铺：

```yaml
source_coordination:
  store:
     dynamodb:
       table_name: "DataPrepperSourceCoordinationStore"
       region: "us-east-1"
       sts_role_arn: "arn:aws:iam::##########:role/SourceCoordinationTableRole"
       ttl: "P7D"
       skip_table_creation: true
```

#### 在-内存商店（默认）

以下示例显示了`in_memory` 商店，最好与单个一起使用-节点群集：


```yaml
source_coordination:
  store:
    in_memory:
```


## 指标

源协调指标的解释不同，具体取决于配置的源。源协调度量的格式为`<sub-pipeline-name>_source_coordinator_<metric-name>`。您可以使用子-管道名称以识别这些指标的来源，因为每个子-管道是每个来源独有的。

### 进度指标

以下是与分区进度有关的指标：

*`partitionsCreatedCount`：已创建的分区项数量。对于S3扫描，这是为其创建分区的对象数量。
*`partitionsCompleted`：已完全处理和标记为`COMPLETED`。对于S3扫描，这是已处理的对象数量。
*`noPartitionsAcquired`：节点试图获取执行工作的分区的次数，但在商店中没有发现可用的分区。用它来表明没有更多数据进入源。
*`partitionsAcquired`：由可以在其上进行工作的节点获取的分区数。在非-错误方案，这应该等于创建的分区数量。
*`partitionsClosed`：已标记为`CLOSED`。这仅适用于使用封闭功能的来源。

以下是与分区错误有关的指标：

*`partitionNotFoundErrors`：表示由节点积极拥有的分区项目没有相应的存储项目。只有在手动删除表中的一个项目时才会发生这种情况。
*`partitionNotOwnedErrors`：表示拥有一个分区的节点由于分区所有权超时到期而失去了所有权。除非来源能够检查点`saveState`，此错误导致重复的项目处理。
*`partitionUpdateErrors`：当该分区项目更新到商店时收到的错误数量失败。有一个前缀`saveState`，`close`， 或者`complete` 指示哪些更新操作失败。



---
layout: default
title: 创建一个集群
nav_order: 1
nav_exclude: true
redirect_from: 
  - /opensearch/cluster/
  - /tuning-your-cluster/cluster/
---

# 创建一个集群

在潜入OpenSearch，搜索和汇总数据之前，首先需要创建一个OpenSearch集群。

OpenSearch可以单一操作-节点或多-节点群集。一般而言，配置两者的步骤非常相似。此页面演示了如何创建和配置多个-节点群集，但只有一些次要调整，您可以按照相同的步骤来创建一个单一的步骤-节点群集。

为了根据您的要求创建和部署OpenSearch群集，重要的是要了解节点发现和群集形成如何工作以及设置如何控制它们。

设计集群有很多方法。下图显示了一个基本体系结构，其中包括四个-节点群集具有一个专用的群集管理器节点，一个专用的协调节点和两个符合群集管理器的数据节点，并且还用于摄入数据。

  主节点最近更改了命名法；现在称为群集管理器节点。
   {： 。笔记 }

![多-节点群集体系结构图]({{site.url}}{{site.baseurl}}/images/cluster.png)

### 节点

下表提供了节点类型的简要说明：

节点类型| 描述| 生产最佳实践
：--- | ：--- | ：-- |
集群管理器| 管理集群的整体操作，并跟踪群集状态。这包括创建和删除索引，跟踪连接并离开群集的节点，检查群集中每个节点的健康状况（通过运行ping请求）以及将碎片分配给节点。| 三个不同区域中的三个专用集群经理节点是几乎所有生产用例的正确方法。这种配置可确保您的群集永远不会失去法定人数。在大多数情况下，两个节点在大部分时间内都会闲置，除非一个节点下降或需要一些维护。
集群经理符合条件| 通过投票过程，选举一个节点作为集群经理节点。| 对于生产群集，请确保您拥有专用的集群管理器节点。实现专用节点类型的方法是将所有其他节点类型标记为false。在这种情况下，您必须将所有其他节点标记为不合格的群集管理器。
数据| 存储和搜索数据。执行所有数据-本地碎片上的相关操作（索引，搜索，汇总）。这些是群集的工作节点，并且需要比任何其他节点类型都多的磁盘空间。| 添加数据节点时，请在区域之间保持平衡。例如，如果您有三个区域，请在三个区域中添加数据节点，每个区域一个。我们建议使用存储和RAM-沉重的节点。
摄取| pre-将数据存储在群集中之前，请处理数据。运行摄入的管道，在将数据添加到索引之前会转换您的数据。| 如果您打算摄入大量数据并运行复杂的摄入管道，我们建议您使用专用的摄入节点。您还可以选择从数据节点中卸载索引，以便将数据节点专门用于搜索和聚合。
协调| 将客户端请求委托给数据节点上的碎片，将结果收集并汇总为一个最终结果，然后将此结果发送回客户端。| 几个专门的协调-只有节点适合防止瓶颈进行搜索-沉重的工作量。我们建议将CPU与尽可能多的内核一起使用。
动态的| 将自定义工作的特定节点委托，例如机器学习（ML）任务，以防止数据节点从数据节点中消耗资源，因此不会影响任何opensearch功能。
搜索| 提供访问权限[可搜索的快照]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/searchable_snapshot/)。合并技术，例如经常使用的段段，并删除使用最少的数据段，以访问可搜索的快照索引（存储在远程长期中-术语存储源，例如Amazon S3或Google Cloud Storage）。| 搜索节点包含一个分配为快照缓存的索引。因此，我们建议使用具有更多计算（CPU和内存）设置的专用节点（与存储容量（硬盘））。

默认情况下，每个节点都是群集-经理-符合条件，数据，摄取和协调节点。决定节点的数量，分配节点类型并为每个节点类型选择硬件取决于您的用例。您必须考虑到想要保留数据的时间，文档的平均大小，典型的工作量（索引，搜索，聚合），预期价格等因素-性能比率，风险承受能力等。

评估所有这些要求后，我们建议您使用基准测试工具[OpenSearch基准测试](https://github.com/opensearch-project/opensearch-benchmark) 提供一个小样本群集并运行具有不同工作负载和配置的测试。比较和分析这些测试的系统和查询指标，以设计最佳体系结构。

此页面演示了如何与不同的节点类型一起使用。它假设你有四个-节点群集类似于前面的插图。

## 先决条件

在开始之前，必须在所有节点上安装和配置Opensearch。有关可用选项的信息，请参阅[安装和配置OpenSearch]({{site.url}}{{site.baseurl}}/opensearch/install/)。

完成后，使用SSH连接到每个节点，然后打开`config/opensearch.yml` 文件。您可以在此文件中为群集设置所有配置。

## 步骤1：命名集群

为集群指定唯一名称。如果您不指定群集名称，则将其设置为`opensearch` 默认情况下。设置描述性群集名称很重要，尤其是如果要在单个网络中运行多个群集时。

要指定群集名称，请更改以下行：

```yml
#cluster.name: my-application
```

到

```yml
cluster.name: opensearch-cluster
```

对所有节点进行相同的更改，以确保它们将加入以形成集群。

## 步骤2：在集群中设置每个节点的节点属性

命名群集后，为群集中的每个节点设置节点属性。

#### 集群管理器节点

给您的群集管理器节点一个名称。如果您没有指定名称，则OpenSearch分配机器-生成的名称，使节点难以监视和故障排除。

```yml
node.name: opensearch-cluster_manager
```

您还可以明确指定此节点是群集管理器节点，即使默认情况下它已经设置为true。将节点角色设置为`cluster_manager` 为了使识别群集管理节点更容易。

```yml
node.roles: [ cluster_manager ]
```

#### 数据节点

将两个节点的名称更改为`opensearch-d1` 和`opensearch-d2`， 分别：

```yml
node.name: opensearch-d1
```

```yml
node.name: opensearch-d2
```

你可以让他们群-经理-合格的数据节点也将用于摄入数据：

```yml
node.roles: [ data, ingest ]
```

您还可以指定要为数据节点设置的任何其他属性。

#### 协调节点

将协调节点的名称更改为`opensearch-c1`：

```yml
node.name: opensearch-c1
```

每个节点默认为一个协调节点，因此要使该节点成为专用的协调节点，`node.roles` 到空列表：

```yml
node.roles: []
```

## 步骤3：将群集绑定到特定的IP地址

`network.bind_host` 定义用于绑定节点的IP地址。默认情况下，OpenSearch在本地主机上听，该主机将群集限制为单个节点。您也可以使用`_local_` 和`_site_` 绑定到任何回环或网站-本地地址，无论是IPv4还是IPv6：

```yml
network.bind_host: [_local_, _site_]
```

形成多个-节点群集，指定节点的IP地址：

```yml
network.bind_host: <IP address of the node>
```

确保在所有节点上配置这些设置。

## 步骤4：为集群配置发现主机

现在，您已经配置了网络主机，您需要配置Discovery Hosts。

禅宗发现是建造的-在，使用的默认机制[单播](https://en.wikipedia.org/wiki/Unicast) 在集群中找到其他节点。

您通常可以添加所有群集-经理-合格的节点`discovery.seed_hosts` 大批。当节点启动时，它会找到另一个群集-经理-合格的节点，确定哪个是群集管理器，并要求加入集群。

例如，对于`opensearch-cluster_manager` 这条线看起来像这样：

```yml
discovery.seed_hosts: ["<private IP of opensearch-d1>", "<private IP of opensearch-d2>", "<private IP of opensearch-c1>"]
```

## 步骤5：开始群集

设置配置后，请在所有节点上启动OpenSearch：

```bash
sudo systemctl start opensearch.service
```

从tar存档安装openSearch不会自动创建一个使用`systemd`。看[运行OpenSearch作为SystemD的服务]({{site.url}}{{site.baseurl}}/opensearch/install/tar/#run-opensearch-as-a-service-with-systemd) 有关如何创建和启动服务的说明，如果您收到错误`Failed to start opensearch.service: Unit not found.`
{： 。提示}

然后转到日志文件以查看集群的形成：

```bash
less /var/log/opensearch/opensearch-cluster.log
```

执行以下操作`_cat` 查询任何节点以查看所有节点形成的群集：

```bash
curl -XGET https://<private-ip>:9200/_cat/nodes?v -u 'admin:admin' --insecure
```

```
ip             heap.percent ram.percent cpu load_1m load_5m load_15m node.role cluster_manager name
x.x.x.x           13          61   0    0.02    0.04     0.05 mi        *      opensearch-cluster_manager
x.x.x.x           16          60   0    0.06    0.05     0.05 md        -      opensearch-d1
x.x.x.x           34          38   0    0.12    0.07     0.06 md        -      opensearch-d2
x.x.x.x           23          38   0    0.12    0.07     0.06 md        -      opensearch-c1
```

为了更好地理解和监视您的群集，请使用[猫API]({{site.url}}{{site.baseurl}}/opensearch/catapis/)。

## （高级）步骤6：配置碎片分配意识或强迫意识

### 碎片分配意识

如果您的节点分布在几个地理区域中，则可以配置碎片分配意识，以将所有副本碎片分配给与主要碎片不同的区域。

有了碎片分配的意识，如果您的一个区域中的节点失败，则可以放心，您的副本碎片散布在其他区域中。它添加了一层容错，以确保您的数据不仅仅是单个节点故障，还可以在区域故障中幸存下来。

要配置碎片分配意识，请将区域属性添加到`opensearch-d1` 和`opensearch-d2`， 分别：

```yml
node.attr.zone: zoneA
```

```yml
node.attr.zone: zoneB
```

更新集群设置：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "zone"
  }
}
```

您可以使用`persistent` 或者`transient` 设置。我们建议`persistent` 设置是因为它通过群集重新启动持续存在。瞬态设置不会通过群集重新启动持续存在。

碎片分配意识试图将多个区域的主要和复制碎片分开。但是，如果仅一个区域可用（例如区域故障后），则将搜索分配给唯一剩余区域。

### 强迫意识

另一个选择是要求将主和复制碎片从不分配给同一区域。这称为强制意识。

要配置强制意识，请为您的区域属性指定所有可能的值：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "zone",
    "cluster.routing.allocation.awareness.force.zone.values":["zoneA", "zoneB"]
  }
}
```

现在，如果数据节点失败，则强制意识不会将副本分配给同一区域中的节点。取而代之的是，群集进入黄色状态，仅当其他区域中的节点在线时才能分配复制品。

在我们的两个-区域架构，如果我们可以使用分配意识`opensearch-d1` 和`opensearch-d2` 使用少于50％，因此每个人都有在同一区域分配复制品的存储能力。
如果不是这样，`opensearch-d1` 和`opensearch-d2` 没有能力包含所有主要和副本碎片，我们可以使用强迫意识。这种方法有助于确保，如果发生故障，OpenSearch不会因缺乏存储而锁定剩余区域并锁定群集。

选择分配意识或强迫意识取决于您在每个区域中可能需要多少空间来平衡主要和复制碎片。

### 副本计数执行

为了强制在所有区域中均匀分布并避免热点，您可以设置`routing.allocation.awareness.balance` 属性为`true`。可以在openSearch.yml文件中配置此设置，并使用cluster Update设置API进行动态更新：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "routing.allocation.awareness.balance": "true"
    }
  }
}
```

这`routing.allocation.awareness.balance` 设置默认情况下是错误的。当它设置为`true`，该索引的碎片总数必须是任何意识属性的最高计数的倍数。例如，考虑具有两个意识属性的配置＆mdash; sones and机架ID。假设有两个区域和三个机架ID。区域数量或机架ID数的最高计数为三个。因此，碎片的数量必须是三个的倍数。如果不是这样，OpenSearch会引发验证异常。

`routing.allocation.awareness.balance` 只有在`cluster.routing.allocation.awareness.attributes` 和`cluster.routing.allocation.awareness.force.zone.values` 设置。
{： 。笔记}

`routing.allocation.awareness.balance` 适用于所有创建或更新索引的操作。例如，假设您在一个区域中运行一个带有三个节点和三个区域的群集-意识设置。如果您尝试使用一个副本创建索引或将索引的设置更新为一个副本，则尝试在验证异常的情况下尝试失败，因为碎片数必须是三个中的倍数。同样，如果您尝试使用一个碎片而没有副本创建一个索引模板，则由于相同的原因，尝试将失败。但是，在所有这些操作中，如果将碎片的数量设置为一个，将复制品数设置为两个，则碎片总数为三个，尝试将成功。

## （高级）步骤7：设置热-温暖的建筑

您可以设计热-温暖的体系结构，您首先将数据索引到热节点---快速而昂贵---一段时间后，将它们移至温暖的节点---缓慢而便宜。

如果分析时间-您很少更新并希望较旧的数据进入更便宜的存储，此体系结构很合适。

这种体系结构有助于节省存储成本的资金。与其增加热节点的数量并使用快速，昂贵的存储空间，还可以为无法频繁访问的数据添加温暖的节点。

配置热-温暖的存储架构，添加`temp` 属性为`opensearch-d1` 和`opensearch-d2`， 分别：

```yml
node.attr.temp: hot
```

```yml
node.attr.temp: warm
```

您可以将属性名称和价值设置为所需的任何内容，只要它对于所有热和温暖的节点都是一致的。

添加索引`newindex` 到热节点：

```json
PUT newindex
{
  "settings": {
    "index.routing.allocation.require.temp": "hot"
  }
}
```

查看以下碎片分配`newindex`：

```json
GET _cat/shards/newindex?v
index     shard prirep state      docs store ip         node
new_index 2     p      STARTED       0  230b 10.0.0.225 opensearch-d1
new_index 2     r      UNASSIGNED
new_index 3     p      STARTED       0  230b 10.0.0.225 opensearch-d1
new_index 3     r      UNASSIGNED
new_index 4     p      STARTED       0  230b 10.0.0.225 opensearch-d1
new_index 4     r      UNASSIGNED
new_index 1     p      STARTED       0  230b 10.0.0.225 opensearch-d1
new_index 1     r      UNASSIGNED
new_index 0     p      STARTED       0  230b 10.0.0.225 opensearch-d1
new_index 0     r      UNASSIGNED
```

在此示例中，所有主要碎片均分配给`opensearch-d1`，这是我们的热节点。所有复制片都没有分配，因为我们强迫此索引仅分配给热节点。

添加索引`oldindex` 到温暖的节点：

```json
PUT oldindex
{
  "settings": {
    "index.routing.allocation.require.temp": "warm"
  }
}
```

碎片分配`oldindex`：

```json
GET _cat/shards/oldindex?v
index     shard prirep state      docs store ip        node
old_index 2     p      STARTED       0  230b 10.0.0.74 opensearch-d2
old_index 2     r      UNASSIGNED
old_index 3     p      STARTED       0  230b 10.0.0.74 opensearch-d2
old_index 3     r      UNASSIGNED
old_index 4     p      STARTED       0  230b 10.0.0.74 opensearch-d2
old_index 4     r      UNASSIGNED
old_index 1     p      STARTED       0  230b 10.0.0.74 opensearch-d2
old_index 1     r      UNASSIGNED
old_index 0     p      STARTED       0  230b 10.0.0.74 opensearch-d2
old_index 0     r      UNASSIGNED
```

在这种情况下，所有主要碎片都分配给`opensearch-d2`。同样，所有复制片都没有分配，因为我们只有一个温暖的节点。

一种流行的方法是配置您的[索引模板]({{site.url}}{{site.baseurl}}/opensearch/index-templates/) 设置`index.routing.allocation.require.temp` 价值`hot`。这样，OpenSearch将您的最新数据存储在热点上。

然后您可以使用[索引状态管理（ISM）]({{site.url}}{{site.baseurl}}/im-plugin/) 插件定期检查索引的年龄并指定操作以采取该索引。例如，当索引达到特定年龄时，请更改`index.routing.allocation.require.temp` 设置为`warm` 要自动将数据从热节点移动到温暖的节点。

## 下一步

如果您使用的是安全插件，则先前的请求`_cat/nodes?v` 可能因初始化错误而失败。有关使用安全插件的完整指导，请参见[安全配置]({{site.url}}{{site.baseurl}}/security/configuration/index/)。


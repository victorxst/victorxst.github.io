---
layout: default
title: 可搜索的快照
parent: 快照
nav_order: 40
grand_parent: 可用性和恢复
redirect_from: 
  - /opensearch/snapshots/searchable_snapshot/
---

# 可搜索的快照

可搜索的快照索引读取数据[快照存储库]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore/#register-repository) 实时（搜索时间），而不是在恢复时间下载所有索引数据以群集存储。由于索引数据保留在存储库中的快照格式中，因此可搜索的快照索引固有地读取-仅有的。任何写入可搜索快照索引的尝试都会导致错误。

可搜索的快照功能包含了诸如集群节点中经常使用的数据段的技术，并从集群节点中删除了最不使用的数据段，以为常用的数据段提供空间。从块存储上的快照下载的数据段与群集节点的一般索引并存。这样，群集节点的计算能力在索引，本地搜索和数据段之间共享，该快照位于较低的快照上-成本对象存储，例如亚马逊简单存储服务（Amazon S3）。尽管更有效地利用了群集节点资源，但大量任务会导致快速搜索较慢。节点的本地存储也用于缓存快照数据。

## 配置一个节点以使用可搜索的快照

要配置可搜索的快照功能，请在opensearch.yml文件中创建一个节点，然后将节点角色定义为`search`：

```yaml
node.name: snapshots-node
node.roles: [ search ]
```

如果您正在运行Docker，则可以使用`search` 通过添加行来节点角色`- node.roles=search` 给你`docker-compose.yml` 文件：

```yaml
version: '3'
services:
  opensearch-node1:
    image: opensearchproject/opensearch:2.7.0
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node1
      - node.roles=search
```

## 创建可搜索的快照索引

通过指定可搜索的快照索引`remote_snapshot` 使用的存储类型使用[还原快照API]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore/#restore-snapshots)。

请求字段| 描述
:--- | :---
`storage_type` | `local` 表示所有快照元数据和索引数据将下载到本地存储。<br /> <br>`remote_snapshot` 表示快照元数据将下载到集群中，但远程存储库将仍然是索引数据的权威存储。数据将根据需要下载和缓存以进行服务查询。群集中的至少一个节点必须配置`search` 节点角色以使用该快照来使用`remote_snapshot` 类型。<br /> <br>默认为`local`。

## 清单索引

要确定索引是否是可搜索的快照索引，请查找具有值的商店类型`remote_snapshot`：

```
GET /my-index/_settings?pretty
```

```json
{
  "my-index": {
    "settings": {
      "index": {
        "store": {
          "type": "remote_snapshot"
        }
      }
    }
  }
}
```

## 潜在用例

以下是可搜索快照功能的潜在用例：

- 从集群中卸载索引的能力-基于存储，但保留了搜索它们的能力。
- 较低的可搜索索引的能力-成本媒体。

## 已知限制

以下是可搜索快照功能的已知局限性：

- 从远程存储库中访问数据的速度比本地磁盘读取要慢，因此预期搜索查询的延迟更高。
- 许多远程对象存储以每项收费-请求检索基础，因此用户应密切监视发生的任何费用。
- 搜索远程数据可能会影响在同一节点上运行的其他查询的性能。我们建议用户提供专用节点`search` 表演的角色-关键应用。


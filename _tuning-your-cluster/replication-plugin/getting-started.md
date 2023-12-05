---
layout: default
title: 入门
nav_order: 15
parent: 跨集群复制
redirect_from:
  - /replication-plugin/get-started/
---

# 开始十字架-集群复制

与十字架-群集复制，您将数据索引到领导者索引，然后将数据复制到一个或多个读取-只有追随者索引。对领导者的所有后续操作将在追随者上复制，例如创建，更新或删除文档。

## 先决条件

叉-群集复制具有以下先决条件：
- 领导者和追随者集群都必须安装复制插件。
- 如果你覆盖了`node.roles` 在`opensearch.yml` 在追随者集群上，请确保它还包括`remote_cluster_client` 角色：

   ```yaml
   node.roles: [<other_roles>, remote_cluster_client]
   ```

## 权限

确保在两个群集上都启用了两个群集或禁用的安全插件。如果禁用了安全插件，则可以跳过此部分。但是，我们强烈建议在生产方案中启用安全插件。

如果启用了安全插件，请确保-管理用户被映射到适当的权限，以便他们可以执行复制操作。对于索引和群集-级别许可要求，请参阅[叉-集群复制权限]({{site.url}}{{site.baseurl}}/replication-plugin/permissions/)。

此外，在领导者群集上验证并添加每个追随者群集节点的杰出名称（DNS），以允许从关注者到领导者的连接。

首先，从每个追随者群集中获取节点的DN：

  ```bash
卷曲-XGET-k-u'Admin：admin''https：// localhost：9200/_opendistro/_security/api/ssl/certs？Pretty'

{
   "transport_certificates_list"：[[
      {
         "issuer_dn" ："CN=Test,OU=Server CA 1B,O=Test,C=US"，
         "subject_dn" ："CN=follower.test.com"，# 要在领导者的nodes_dn配置下添加
         "not_before" ："2021-11-12T00:00:00Z"，
         "not_after" ："2022-12-11T23:59:59Z"
      }
   这是给出的
}
  ```

Then verify that it's part of the leader cluster configuration in `opensearch.yml`. Otherwise, add it under the following setting:

  ```yaml
plugins.security.nodes_dn：
  - "CN=*.leader.com, OU=SSL, O=Test, L=Test, C=DE" # 已经配置的一部分
  - "CN=follower.test.com" # 从以上来自追随者的回应
  ```
## Example setup

To start two single-node clusters on the same network, save this sample file as `Docker-compose.yml` and run `Docker-组成`:

```yml
version: '3'
services:
  replication-node1:
    image: opensearchproject/opensearch:{{site.opensearch_version}}
    container_name: replication-node1
    environment:
      - cluster.name=leader-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
    ports:
      - 9201:9200
      - 9700:9600 # required for Performance Analyzer
    networks:
      - opensearch-net
  replication-node2:
    image: opensearchproject/opensearch:{{site.opensearch_version}}
    container_name: replication-node2
    environment:
      - cluster.name=follower-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - opensearch-data1:/usr/share/opensearch/data
    ports:
      - 9200:9200
      - 9600:9600 # required for Performance Analyzer
    networks:
      - opensearch-net

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```

After the clusters start, verify the names of each:

```bash
curl -XGET -u 'admin:admin' -k 'https://localhost:9201'
{
  "cluster_name" : "leader-cluster",
  ...
}

curl -XGET -u 'admin:admin' -k 'https://localhost:9200'
{
  "cluster_name" : "follower-cluster",
  ...
}
```

For this example, use port 9201 (`复制-Node1`) as the leader and port 9200 (`复制-Node2`）作为追随者集群。

要获取领导者群集的IP地址，请首先确定其容器ID：

```bash
docker ps
CONTAINER ID    IMAGE                                       PORTS                                                      NAMES
3b8cdc698be5    opensearchproject/opensearch:{{site.opensearch_version}}   0.0.0.0:9200->9200/tcp, 0.0.0.0:9600->9600/tcp, 9300/tcp   replication-node2
731f5e8b0f4b    opensearchproject/opensearch:{{site.opensearch_version}}   9300/tcp, 0.0.0.0:9201->9200/tcp, 0.0.0.0:9700->9600/tcp   replication-node1
```

然后获取该容器的IP地址：

```bash
docker inspect --format='{% raw %}{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}{% endraw %}' 731f5e8b0f4b
172.22.0.3
```

## 建立十字架-集群连接

叉-群集复制遵循"pull" 模型，因此大多数更改发生在追随者群集上，而不是领导者群集。

在跟随器群集上，为每个种子节点添加IP地址（带有端口9300）。因为这是一个-节点群集，您只有一个种子节点。提供连接的描述性名称，您将在请求中使用该连接：开始复制：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_cluster/settings?pretty' -d '
{
  "persistent": {
    "cluster": {
      "remote": {
        "my-connection-alias": {
          "seeds": ["172.22.0.3:9300"]
        }
      }
    }
  }
}'
```

## 开始复制

要开始，创建一个称为的索引`leader-01` 在领导者集群上：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9201/leader-01?pretty'
```

然后从追随者群集开始复制。在请求主体中，提供您要复制的连接名称和领导索引以及要使用的安全角色：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_start?pretty' -d '
{
   "leader_alias": "my-connection-alias",
   "leader_index": "leader-01",
   "use_roles":{
      "leader_cluster_role": "all_access",
      "follower_cluster_role": "all_access"
   }
}'
```

如果禁用了安全插件，请省略`use_roles` 范围。但是，如果启用了它，则必须指定OpenSearch用来对请求进行身份验证的领导者和追随者集群角色。此示例使用`all_access` 为简单起见，但是我们建议在每个群集上创建复制用户，并且[相应地绘制它]({{site.url}}{{site.baseurl}}/replication-plugin/permissions/#map-the-leader-and-follower-cluster-roles)。
{: .tip }

此命令创建了相同的读取-只有索引命名`follower-01` 在不断更新的追随者群集上，随着更改的更改`leader-01` 领导者集群的索引。启动复制从头开始创建一个追随者索引-- 您无法将现有索引转换为追随者索引。

## 确认复制

复制开始后，获得状态：

```bash
curl -XGET -k -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_status?pretty'

{
  "status" : "SYNCING",
  "reason" : "User initiated",
  "leader_alias" : "my-connection-alias",
  "leader_index" : "leader-01",
  "follower_index" : "follower-01",
  "syncing_details" : {
    "leader_checkpoint" : -1,
    "follower_checkpoint" : -1,
    "seq_no" : 0
  }
}
```

可能的状态是`SYNCING`，`BOOTSTRAPPING`，`PAUSED`， 和`REPLICATION NOT IN PROGRESS`。

领导者和追随者检查点值以负数开始并反映碎片计数（-1碎片，-5对于五片，依此类推）。每次更改的值会增加，并说明追随者在领导者背后有多少更新。如果索引完全同步，则值相同。

要确认复制实际上正在发生，请在领导者索引中添加文档：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9201/leader-01/_doc/1?pretty' -d '{"The Shining": "Stephen King"}'
```

然后验证追随者索引上的复制内容：

```bash
curl -XGET -k -u 'admin:admin' 'https://localhost:9200/follower-01/_search?pretty'

{
  ...
  "hits": [{
    "_index": "follower-01",
    "_id": "1",
    "_score": 1.0,
    "_source": {
      "The Shining": "Stephen King"
    }
  ]
}
```

## 暂停和简历复制

如果需要补救问题或减少领导者集群的负载，则可以暂时暂停索引复制：

```bash
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_pause?pretty' -d '{}'
```

要确认复制已暂停，请获得状态：

```bash
curl -XGET -k -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_status?pretty'

{
  "status" : "PAUSED",
  "reason" : "User initiated",
  "leader_alias" : "my-connection-alias",
  "leader_index" : "leader-01",
  "follower_index" : "follower-01"
}
```

完成更改后，恢复复制：

```bash
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_resume?pretty' -d '{}'
```

复制恢复时，追随者索引在暂停复制时会挑选对领导者索引的任何更改。

请注意，在暂停超过12个小时后，您无法恢复复制。你必须[停止复制]({{site.url}}{{site.baseurl}}/replication-plugin/api/#stop-replication)，删除追随者索引，然后重新启动领导者的复制。

## 停止复制

当您不再需要复制索引时，请从追随者群集终止复制：

```bash
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_stop?pretty' -d '{}'
```

当您停止复制时，追随者索引联合国-跟随领导者，成为您可以写入的标准索引。停止后，您无法重新启动复制。

获取状态以确认该索引不再被复制：

```bash
curl -XGET -k -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/follower-01/_status?pretty'

{
  "status" : "REPLICATION NOT IN PROGRESS"
}
```

您可以进一步确认通过对领导者索引进行修改并确认他们不会出现在追随者索引上，从而停止了复制。




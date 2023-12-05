---
layout: default
title: 跨集群搜索
parent: 访问控制
nav_order: 105
redirect_from:
 - /security/access-control/cross-cluster-search/
 - /security-plugin/access-control/cross-cluster-search/
---

# 跨集群搜索

跨集群搜索正是听起来的样子：它可以让群集中的任何节点在其他群集上执行搜索请求。安全插件支持交叉-集群搜索开箱即用。

---

#### Table of contents
1. TOC
{:toc}


---

## 身份验证流

从一个 *协调群集 *访问 *远程群集 *时 *使用Cross *-集群搜索：

1. 安全插件在协调集群上对用户进行身份验证。
1. 安全插件在协调群集上获取用户的后端角色。
1. 包括身份验证的用户在内的呼叫已转发到远程群集。
1. 用户的权限在远程集群上评估。

您可以在远程和协调集群上具有不同的身份验证和授权配置，但是我们建议在这两个上使用相同的设置。


## 权限

要查询远程簇的索引，用户需要`READ` 或者`SEARCH` 权限。此外，搜索请求包括查询参数`ccs_minimize_roundtrips=false`  - 告诉OpenSearch不要最大程度地减少向远程群集的传出和挖掘请求 - 用户需要对索引获得以下额外权限：

```
indices:admin/shards/search_shards
```

有关有关的更多信息`ccs_minimize_roundtrips` 参数，请参阅列表[URL参数]({{site.url}}{{site.baseurl}}/api-reference/search/#url-parameters) 对于搜索API。

#### 样本角色。配置

```yml
humanresources:
  cluster:
    - CLUSTER_COMPOSITE_OPS_RO
  indices:
    'humanresources':
      '*':
        - READ
        - indices:admin/shards/search_shards # needed when the search request includes parameter setting 'ccs_minimize_roundtrips=false'.
```


#### OpenSearch仪表板中的样本角色

![OpenSearch仪表板UI用于创建十字架-集群搜索角色]({{site.url}}{{site.baseurl}}/images/security-ccs.png)


## 演练

将此文件另存为`docker-compose.yml` 并运行`docker-compose up` 开始两个单身-同一网络上的节点簇：

```yml
version: '3'
services:
  opensearch-ccs-node1:
    image: opensearchproject/opensearch:{{site.opensearch_version}}
    container_name: opensearch-ccs-node1
    environment:
      - cluster.name=opensearch-ccs-cluster1
      - discovery.type=single-node
      - bootstrap.memory_lock=true # along with the memlock settings below, disables swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
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

  opensearch-ccs-node2:
    image: opensearchproject/opensearch:{{site.opensearch_version}}
    container_name: opensearch-ccs-node2
    environment:
      - cluster.name=opensearch-ccs-cluster2
      - discovery.type=single-node
      - bootstrap.memory_lock=true # along with the memlock settings below, disables swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
    ports:
      - 9250:9200
      - 9700:9600 # required for Performance Analyzer
    networks:
      - opensearch-net

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```

簇开始后，验证每个名称：

```json
curl -XGET -u 'admin:admin' -k 'https://localhost:9200'
{
  "cluster_name" : "opensearch-ccs-cluster1",
  ...
}

curl -XGET -u 'admin:admin' -k 'https://localhost:9250'
{
  "cluster_name" : "opensearch-ccs-cluster2",
  ...
}
```

两个簇都在`localhost`，因此重要的标识符是端口号。在这种情况下，使用端口9200（`opensearch-ccs-node1`）作为远程群集和端口9250（`opensearch-ccs-node2`）作为协调集群。

要获取远程群集的IP地址，请首先确定其容器ID：

```bash
docker ps
CONTAINER ID    IMAGE                                       PORTS                                                      NAMES
6fe89ebc5a8e    opensearchproject/opensearch:{{site.opensearch_version}}   0.0.0.0:9200->9200/tcp, 0.0.0.0:9600->9600/tcp, 9300/tcp   opensearch-ccs-node1
2da08b6c54d8    opensearchproject/opensearch:{{site.opensearch_version}}   9300/tcp, 0.0.0.0:9250->9200/tcp, 0.0.0.0:9700->9600/tcp   opensearch-ccs-node2
```

然后获取该容器的IP地址：

```bash
docker inspect --format='{% raw %}{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}{% endraw %}' 6fe89ebc5a8e
172.31.0.3
```

在协调集群上，为每个添加远程群集名称和IP地址（带有端口9300）"seed node." 在这种情况下，您只有一个种子节点：

```json
curl -k -XPUT -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9250/_cluster/settings' -d '
{
  "persistent": {
    "cluster.remote": {
      "opensearch-ccs-cluster1": {
        "seeds": ["172.31.0.3:9300"]
      }
    }
  }
}'
```

在远程集群上，索引一个文档：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/books/_doc/1' -d '{"Dracula": "Bram Stoker"}'
```

此时，交叉-集群搜索有效。您可以使用`admin` 用户：

```bash
curl -XGET -k -u 'admin:admin' 'https://localhost:9250/opensearch-ccs-cluster1:books/_search?pretty'
{
  ...
  "hits": [{
    "_index": "opensearch-ccs-cluster1:books",
    "_id": "1",
    "_score": 1.0,
    "_source": {
      "Dracula": "Bram Stoker"
    }
  }]
}
```

要继续测试，请在两个群集上创建新用户：

```bash
curl -XPUT -k -u 'admin:admin' 'https://localhost:9200/_plugins/_security/api/internalusers/booksuser' -H 'Content-Type: application/json' -d '{"password":"password"}'
curl -XPUT -k -u 'admin:admin' 'https://localhost:9250/_plugins/_security/api/internalusers/booksuser' -H 'Content-Type: application/json' -d '{"password":"password"}'
```

然后进行与以前相同的搜索`booksuser`：

```json
curl -XGET -k -u booksuser:password 'https://localhost:9250/opensearch-ccs-cluster1:books/_search?pretty'
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "no permissions for [indices:admin/shards/search_shards, indices:data/read/search] and User [name=booksuser, roles=[], requestedTenant=null]"
      }
    ],
    "type" : "security_exception",
    "reason" : "no permissions for [indices:admin/shards/search_shards, indices:data/read/search] and User [name=booksuser, roles=[], requestedTenant=null]"
  },
  "status" : 403
}
```

注意权限错误。在远程集群上，使用适当的权限创建角色，然后映射`booksuser` 担任这个角色：

```bash
curl -XPUT -k -u 'admin:admin' -H 'Content-Type: application/json' 'https://localhost:9200/_plugins/_security/api/roles/booksrole' -d '{"index_permissions":[{"index_patterns":["books"],"allowed_actions":["indices:admin/shards/search_shards","indices:data/read/search"]}]}'
curl -XPUT -k -u 'admin:admin' -H 'Content-Type: application/json' 'https://localhost:9200/_plugins/_security/api/rolesmapping/booksrole' -d '{"users" : ["booksuser"]}'
```

两个群集必须具有用户，但是只有远程群集需要角色和映射。在这种情况下，协调集群处理身份验证（即"Does this request include valid user credentials?"），远程集群处理授权（即"Can this user access this data?"）。
{: .tip }

最后，重复搜索：

```bash
curl -XGET -k -u booksuser:password 'https://localhost:9250/opensearch-ccs-cluster1:books/_search?pretty'
{
  ...
  "hits": [{
    "_index": "opensearch-ccs-cluster1:books",
    "_id": "1",
    "_score": 1.0,
    "_source": {
      "Dracula": "Bram Stoker"
    }
  }]
}
```


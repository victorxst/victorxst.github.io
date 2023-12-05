---
layout: default
title: 搜索模型组
parent: 模型组API
grand_parent: ML Commons API
nav_order: 30
---

# 搜索模型组

当您搜索模型组时，只有那些访问访问的模型组将被返回。例如，对于所有查询，将返回的模型组为：

- 索引中的所有公共模型组
- 您是所有者的私人模型组
- 与至少共享一个的模型组`backend_roles` 与你

有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。

## 路径和HTTP方法

```json
POST /_plugins/_ml/model_groups/_search
GET /_plugins/_ml/model_groups/_search
```

#### 示例请求：匹配全部

以下请求由`user1` 谁有`IT` 和`HR` 角色：

```json
POST /_plugins/_ml/model_groups/_search
{
  "query": {
    "match_all": {}
  },
  "size": 1000
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "took": 31,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": ".plugins-ml-model-group",
        "_id": "TRqZfYgBD7s2oEFdvrQj",
        "_version": 1,
        "_seq_no": 2,
        "_primary_term": 1,
        "_score": 1,
        "_source": {
          "backend_roles": [
            "HR",
            "IT"
          ],
          "owner": {
            "backend_roles": [
              "HR",
              "IT"
            ],
            "custom_attribute_names": [],
            "roles": [
              "ml_full_access",
              "own_index",
              "test_ml"
            ],
            "name": "user1",
            "user_requested_tenant": "__user__"
          },
          "created_time": 1685734407714,
          "access": "restricted",
          "latest_version": 0,
          "last_updated_time": 1685734407714,
          "name": "model_group_test",
          "description": "This is an example description"
        }
      },
      {
        "_index": ".plugins-ml-model-group",
        "_id": "URqZfYgBD7s2oEFdyLTm",
        "_version": 1,
        "_seq_no": 3,
        "_primary_term": 1,
        "_score": 1,
        "_source": {
          "backend_roles": [
            "IT"
          ],
          "owner": {
            "backend_roles": [
              "HR",
              "IT"
            ],
            "custom_attribute_names": [],
            "roles": [
              "ml_full_access",
              "own_index",
              "test_ml"
            ],
            "name": "user1",
            "user_requested_tenant": "__user__"
          },
          "created_time": 1685734410470,
          "access": "restricted",
          "latest_version": 0,
          "last_updated_time": 1685734410470,
          "name": "model_group_test",
          "description": "This is an example description"
        }
      },
      ...
    ]
  }
}
```

#### 示例请求：搜索带有所有者名称的模型组

以下要求搜索模型组的请求`user` 发送`user2` 谁有`IT` 后端角色：

```json
GET /_plugins/_ml/model_groups/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "query": {
              "term": {
                "owner.name.keyword": {
                  "value": "user1",
                  "boost": 1
                }
              }
            },
            "path": "owner",
            "ignore_unmapped": false,
            "score_mode": "none",
            "boost": 1
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 0,
    "hits": [
      {
        "_index": ".plugins-ml-model-group",
        "_id": "TRqZfYgBD7s2oEFdvrQj",
        "_version": 1,
        "_seq_no": 2,
        "_primary_term": 1,
        "_score": 0,
        "_source": {
          "backend_roles": [
            "HR",
            "IT"
          ],
          "owner": {
            "backend_roles": [
              "HR",
              "IT"
            ],
            "custom_attribute_names": [],
            "roles": [
              "ml_full_access",
              "own_index",
              "test_ml"
            ],
            "name": "user1",
            "user_requested_tenant": "__user__"
          },
          "created_time": 1685734407714,
          "access": "restricted",
          "latest_version": 0,
          "last_updated_time": 1685734407714,
          "name": "model_group_test",
          "description": "This is an example description"
        }
      },
      ...
    ]
  }
}
```

#### 示例请求：搜索具有模型组ID的模型组

```json
GET /_plugins/_ml/model_groups/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "_id": [
              "HyPNK4gBwNxGowI0AtDk"
            ]
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": ".plugins-ml-model-group",
        "_id": "HyPNK4gBwNxGowI0AtDk",
        "_version": 3,
        "_seq_no": 16,
        "_primary_term": 5,
        "_score": 1,
        "_source": {
          "backend_roles": [
            "IT"
          ],
          "owner": {
            "backend_roles": [
              "",
              "HR",
              "IT"
            ],
            "custom_attribute_names": [],
            "roles": [
              "ml_full_access",
              "own_index",
              "test-ml"
            ],
            "name": "user1",
            "user_requested_tenant": null
          },
          "created_time": 1684362035938,
          "latest_version": 2,
          "last_updated_time": 1684362571300,
          "name": "model_group_test",
          "description": "This is an example description"
        }
      }
    ]
  }
}
```

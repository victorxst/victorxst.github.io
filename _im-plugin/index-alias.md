---
layout: default
title: 索引别名
nav_order: 11
redirect_from:
  - /opensearch/index-alias/
---

# 索引别名

别名是可以指向一个或多个索引的虚拟索引名称。

如果数据分布在多个索引中，则可以创建别名并改为查询它，而不是跟踪要查询的索引。

例如，如果要根据月份将日志存储到索引中，并且经常查询前两个月的日志，则可以创建 `last_2_months` 别名并更新它每月指向的索引。

由于你可以随时更改别名指向的索引，因此在应用程序中使用别名引用索引允许你在不停机的情况下重新索引数据。

---

#### 目录
1. 目录
{:toc}


---

## 创建别名

要创建别名，请使用 POST 请求：

```json
POST _aliases
```

使用该 `actions` 方法指定要执行的操作列表。此命令创建一个名为 `alias1` 别名的别名，并将其添加到 `index-1` 此别名中：

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index-1",
        "alias": "alias1"
      }
    }
  ]
}
```

你应看到以下响应：

```json
{
   "acknowledged": true
}
```

如果此请求失败，请确保要添加到别名的索引已存在。

你还可以使用以下请求之一创建别名：

```json
PUT <index>/_aliases/<alias name>
POST <index>/_aliases/<alias name>
PUT <index>/_alias/<alias name>
POST <index>/_alias/<alias name>
```

 `<index>` 上述请求中的可以是索引名称、以逗号分隔的索引名称列表或通配符表达式。用于 `_all` 引用所有索引。

若要检查是否 `alias1` 引用 `index-1`，请运行以下命令之一：

```json
GET /_alias/alias1
GET /index-1/_alias/alias1
```

若要获取别名引用的索引的映射和设置信息，请运行以下命令：

```json
GET alias1
```

## 添加或删除索引

你可以在同一 `_aliases` 操作中执行多个操作。例如，以下命令删除 `index-1` 并添加到 `index-2` `alias1`：

```json
POST _aliases
{
  "actions": [
    {
      "remove": {
        "index": "index-1",
        "alias": "alias1"
      }
    },
    {
      "add": {
        "index": "index-2",
        "alias": "alias1"
      }
    }
  ]
}
```

 `add` 和动作以原子方式发生，这意味着在任何时候都不会 `alias1` 同时 `index-1` 指向和 `remove` `index-2`。

你还可以根据索引模式添加索引：

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index*",
        "alias": "alias1"
      }
    }
  ]
}
```

## 管理别名

若要列出别名到索引的映射，请运行以下命令：

```json
GET _cat/aliases?v
```

#### 响应示例

```json
alias     index   filter    routing.index   routing.search
alias1    index-1   *             -                 -
```

若要检查别名指向的索引，请运行以下命令：

```json
GET _alias/alias1
```

#### 响应示例

```json
{
  "index-2": {
    "aliases": {
      "alias1": {}
    }
  }
}
```

相反，若要查找指向特定索引的别名，请运行以下命令：

```json
GET /index-2/_alias/*
```

若要获取所有索引名称及其别名，请运行以下命令：

```json
GET /_alias
```

若要检查别名是否存在，请运行以下命令之一：

```json
HEAD /alias1/_alias/
HEAD /_alias/alias1/
HEAD index-1/_alias/alias1/
```

## 在创建索引时添加别名

你可以在创建索引时向别名添加索引：

```json
PUT index-1
{
  "aliases": {
    "alias1": {}
  }
}
```

## 创建过滤的别名

你可以创建筛选的别名，以访问基础索引中的文档或字段的子集。

此命令仅将特定的时间戳字段 `alias1` 添加到：

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index-1",
        "alias": "alias1",
        "filter": {
          "term": {
            "timestamp": "1574641891142"
          }
        }
      }
    }
  ]
}
```

## 索引别名选项

你可以指定下表中显示的选项。

选项 | 有效值 | 描述 | 必填
:--- | :--- | :---
 `index` | 字符串 | 别名指向的索引的名称。| 是的
 `alias` | 字符串 | 别名的名称。| 不
 `filter` | 对象 | 向别名添加过滤器。| 不
 `routing` | 字符串 | 将搜索限制为关联的分片值。你可以单独指定 `search_routing` 和 `index_routing`。| 不
 `is_write_index` | 字符串 | 指定接受对别名执行任何写入操作的索引。如果未指定此值，则不允许执行写入操作。| 不


## 删除别名

若要从索引中删除一个或多个别名，请使用以下请求：

```json
DELETE <index>/_alias/<alias>
DELETE <index>/_aliases/<alias>
```

和 `<alias>` 上述请求中都 `<index>` 支持逗号分隔的列表和通配符表达式。 `<alias>` 用于 `_all` 代替删除中 `<index>` 列出的索引的所有别名。

例如，如果 `alias1` refer to `index-1` 和 `index-2`，则可以运行以下命令从： `alias1` `index-1`

```json
DELETE index-1/_alias/alias1
```

运行上述请求后，不再引用 `index-1`， `alias1` 但仍指 `index-2`。
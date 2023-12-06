---
layout: default
title: 索引模板
nav_order: 6
redirect_from:
  - /opensearch/index-templates/
---

# 索引模板

索引模板允许你使用预定义的映射和设置初始化新索引。例如，如果连续为日志数据编制索引，则可以定义索引模板，以便所有这些索引具有相同数量的分片和副本。

### 创建模板

若要创建索引模板，请使用 PUT 或 POST 请求：

```json
PUT _index_template/<template name>
POST _index_template/<template name>
```

此命令创建一个名为 `daily_logs` 该模板的模板，并将其应用于名称与模式 `logs-2020-01-*` 匹配的任何新索引，并将其添加到别名中 `my_logs`：

```json
PUT _index_template/daily_logs
{
  "index_patterns": [
    "logs-2020-01-*"
  ],
  "template": {
    "aliases": {
      "my_logs": {}
    },
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "value": {
          "type": "double"
        }
      }
    }
  }
}
```

你应看到以下响应：

```json
{
  "acknowledged": true
}
```

如果创建名为 `logs-2020-01-01` 的索引，则可以看到它具有模板中的映射和设置：

```json
PUT logs-2020-01-01
GET logs-2020-01-01
```

```json
{
  "logs-2020-01-01": {
    "aliases": {
      "my_logs": {}
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "value": {
          "type": "double"
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1578107970779",
        "number_of_shards": "2",
        "number_of_replicas": "1",
        "uuid": "U1vMDMOHSAuS2IzPcPHpOA",
        "version": {
          "created": "7010199"
        },
        "provided_name": "logs-2020-01-01"
      }
    }
  }
}
```

与此模式匹配的任何其他索引--- `logs-2020-01-02`、 `logs-2020-01-03` 等---将继承相同的映射和设置。

索引模式不能包含以下任何字符： `:`、、、 `/`、 `+` `#` `\` `<` `"` `|` `?` `>` 和。

### 检索模板

要列出所有索引模板，请执行以下操作：

```json
GET _cat/templates
GET /_index_template
```

要按名称查找模板，请执行以下操作：

```json
GET _index_template/daily_logs
```

要获取与模式匹配的所有模板的列表，请执行以下操作：

```json
GET _index_template/daily*
```

要检查特定模板是否存在，请执行以下操作：

```json
HEAD _index_template/<name>
```

### 配置多个模板

你可以为索引创建多个索引模板。如果索引名称与多个模板匹配，则 OpenSearch 会从优先级最高的模板中获取映射和设置，并将其应用于索引。

例如，假设你有以下两个模板，它们都与 `logs-2020-01-02` 索引匹配，并且字段中存在冲突 `number_of_shards`：

#### 模板 1

```json
PUT _index_template/template-01
{
  "index_patterns": [
    "logs*"
  ],
  "priority": 0,
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 2
    }
  }
}
```

#### 模板 2

```json
PUT _index_template/template-02
{
  "index_patterns": [
    "logs-2020-01-*"
  ],
  "priority": 1,
  "template": {
    "settings": {
      "number_of_shards": 3
    }
  }
}
```

因为 `template-02` 具有更高的 `priority` 值，所以它优先于 `template-01`。 `logs-2020-01-02` 索引的值为 `number_of_shards` 3 `number_of_replicas`，默认值为 1。

### 删除模板

你可以使用索引模板的名称删除索引模板：

```json
DELETE _index_template/daily_logs
```

## 可组合索引模板

管理多个索引模板面临以下挑战：

- 如果索引模板之间存在重复项，则存储这些索引模板会导致更大的群集状态。
- 如果要对所有索引模板进行更改，则必须手动对每个模板进行更改。

你可以使用可组合的索引模板来克服这些挑战。通过可组合索引模板，可以将通用设置、映射和别名抽象为称为组件模板的可重用构建基块。

你可以组合组件模板来组合索引模板。

直接在请求中指定的设置和映射将覆盖索引模板及其组件模板中[创建索引]({{site.url}}{{site.baseurl}}/api-reference/index-apis/create-index/)指定的任何设置或映射。
{: .note}

### 创建组件模板

让我们定义两个组件模板--- `component_template_1` 和 `component_template_2`：

#### 组件模板 1

```json
PUT _component_template/component_template_1
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        }
      }
    }
  }
}
```

#### 组件模板 2

```json
PUT _component_template/component_template_2
{
  "template": {
    "mappings": {
      "properties": {
        "ip_address": {
          "type": "ip"
        }
      }
    }
  }
}
```

### 使用组件模板创建索引模板

创建索引模板时，需要将组件模板 `composed_of` 包含在列表中。

OpenSearch 按照你在索引模板中指定组件模板的顺序应用组件模板。最后应用你在索引模板中指定的设置、映射和别名。

```json
PUT _index_template/daily_logs
{
  "index_patterns": [
    "logs-2020-01-*"
  ],
  "template": {
    "aliases": {
      "my_logs": {}
    },
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "value": {
          "type": "double"
        }
      }
    }
  },
  "priority": 200,
  "composed_of": [
    "component_template_1",
    "component_template_2"
  ],
  "version": 3,
  "_meta": {
    "description": "using component templates"
  }
}
```

如果创建名为 `logs-2020-01-01` 的索引，则可以看到它从两个组件模板派生其映射和设置：

```json
PUT logs-2020-01-01
GET logs-2020-01-01
```

#### 响应示例

```json
{
  "logs-2020-01-01": {
    "aliases": {
      "my_logs": {}
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "ip_address": {
          "type": "ip"
        },
        "timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "value": {
          "type": "double"
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1625382479459",
        "number_of_shards": "2",
        "number_of_replicas": "1",
        "uuid": "rYUlpOXDSUSuZifQLPfa5A",
        "version": {
          "created": "7100299"
        },
        "provided_name": "logs-2020-01-01"
      }
    }
  }
}
```


## 索引模板选项

你可以指定以下模板选项：

选项 | 类型 | 描述 | 必填
:--- | :--- | :--- | :---
 `template` | `Object` | 指定索引设置、映射和别名。| 不
 `priority` | `Integer` | 索引模板的优先级。| 不
 `composed_of` | `String array` | 应用于新索引的组件模板的名称与当前模板一起。| 不
 `version` | `Integer` | 指定版本号以简化模板管理。默认值为 `null`. | 不
 `_meta ` | `Object` | 指定有关模板的元信息。| 不

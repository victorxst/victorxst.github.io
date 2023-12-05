---
layout: default
title: 搜索模板
nav_order: 50
redirect_from:
  - /opensearch/search-template/
---

# 搜索模板

你可以转换你的-文本查询到搜索模板中以接受用户输入并将其动态插入您的查询中。

例如，如果您将OpenSearch用作应用程序或网站的后端搜索引擎，则可以从搜索栏或表单字段中获取用户查询，并将其作为参数传递到搜索模板中。这样，创建OpenSearch查询的语法将从您的最终用户中抽象出来。

当您编写代码以将用户输入转换为OpenSearch查询时，您可以使用搜索模板简化代码。如果您需要在搜索查询中添加字段，则可以在不更改代码的情况下修改模板。

搜索模板使用胡须语言。有关所有语法选项的列表，请参见[胡子手册](https://mustache.github.io/mustache.5.html)。
{： 。笔记 }

## 创建搜索模板

搜索模板具有两个组件：查询和参数。参数是用户-输入的值将放置在变量中。变量在小胡子符号中用双括号表示。遇到变量时`{% raw %}{{var}}{% endraw %}` 在查询中，OpenSearch转到`params` 部分，寻找一个称为的参数`var`，并用指定的值替换。

您可以编码您的应用程序，以询问您的用户要搜索什么，然后将该值插入`params` 在运行时对象。

此命令定义了一个搜索模板以查找播放的名称。这`{% raw %}{{play_name}}{% endraw %}` 在查询中被值代替`Henry IV`：

```json
GET _search/template
{
  "source": {
    "query": {
      "match": {
        "play_name": "{% raw %}{{play_name}}{% endraw %}"
      }
    }
  },
  "params": {
    "play_name": "Henry IV"
  }
}
```

该模板在整个群集上运行搜索。
要在特定索引上运行此搜索，请在请求中添加索引名称：

```json
GET shakespeare/_search/template
```

指定`from` 和`size` 参数：

```json
GET _search/template
{
  "source": {
    "from": "{% raw %}{{from}}{% endraw %}",
    "size": "{% raw %}{{size}}{% endraw %}",
    "query": {
      "match": {
        "play_name": "{% raw %}{{play_name}}{% endraw %}"
      }
    }
  },
  "params": {
    "play_name": "Henry IV",
    "from": 10,
    "size": 10
  }
}
```

为了改善搜索体验，您可以定义默认值，以便用户不必指定所有可能的参数。如果未在`params` 部分，OpenSearch使用默认值。

用于定义变量默认值的语法`var` 如下：

```json
{% raw %}{{var}}{{^var}}default value{{/var}}{% endraw %}
```

此命令为`from` AS 10和`size` AS 10：

```json
GET _search/template
{
  "source": {
    "from": "{% raw %}{{from}}{{^from}}10{{/from}}{% endraw %}",
    "size": "{% raw %}{{size}}{{^size}}10{{/size}}{% endraw %}",
    "query": {
      "match": {
        "play_name": "{% raw %}{{play_name}}{% endraw %}"
      }
    }
  },
  "params": {
    "play_name": "Henry IV"
  }
}
```


## 保存和执行搜索模板

搜索模板按照您想要的方式工作后，您可以将该模板的源保存为脚本，从而使其可重复使用，以适用于不同的输入参数。

将搜索模板保存为脚本时，您需要指定`lang` 参数为`mustache`：

```json
POST _scripts/play_search_template
{
  "script": {
    "lang": "mustache",
    "source": {
      "from": "{% raw %}{{from}}{{^from}}0{{/from}}{% endraw %}",
      "size": "{% raw %}{{size}}{{^size}}10{{/size}}{% endraw %}",
      "query": {
        "match": {
          "play_name": "{{play_name}}"
        }
      }
    },
    "params": {
      "play_name": "Henry IV"
    }
  }
}
```

现在，您可以通过参考其模板重复使用该模板`id` 范围。
您可以将此源模板重复使用不同的输入值。

```json
GET _search/template
{
  "id": "play_search_template",
  "params": {
    "play_name": "Henry IV",
    "from": 0,
    "size": 1
  }
}
```
#### 样本输出

```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 6,
    "successful": 6,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3205,
      "relation": "eq"
    },
    "max_score": 3.641852,
    "hits": [
      {
        "_index": "shakespeare",
        "_type": "_doc",
        "_id": "4",
        "_score": 3.641852,
        "_source": {
          "type": "line",
          "line_id": 5,
          "play_name": "Henry IV",
          "speech_number": 1,
          "line_number": "1.1.2",
          "speaker": "KING HENRY IV",
          "text_entry": "Find we a time for frighted peace to pant,"
        }
      }
    ]
  }
}
```

如果您有存储的模板并想验证它，请使用`render` 手术：

```json
POST _render/template
{
  "id": "play_search_template",
  "params": {
    "play_name": "Henry IV"
  }
}
```

#### 样本输出

```json
{
  "template_output": {
    "from": "0",
    "size": "10",
    "query": {
      "match": {
        "play_name": "Henry IV"
      }
    }
  }
}
```

支持以下渲染操作：

```json
GET /_render/template
POST /_render/template
GET /_render/template/<id>
POST /_render/template/<id>
```

## 使用搜索模板的高级参数转换

您在小胡子中有许多不同的语法选项，可以将输入参数转换为查询。
您可以指定条件，运行循环，加入数组，将阵列转换为JSON等等。

### 状况

在胡须中使用截面标签来表示条件：

```json
{% raw %}{{#var}}var{{/var}}{% endraw %}
```

什么时候`var` 是布尔值，该语法充当`if` 健康）状况。这`{% raw %}{{#var}}{% endraw %}` 和`{% raw %}{{/var}}{% endraw %}` 标签插入仅当它们之间放置的值`var` 评估`true`。

使用节标签将使您的JSON无效，因此您必须以字符串格式编写查询。

此命令包括`size` 仅在查询中的参数`limit` 参数设置为`true`。
在下面的示例中，`limit` 参数为`true`， 所以`size` 参数已激活。结果，您只会撤回两个文档。

```json
GET _search/template
{
  "source": "{% raw %}{ {{#limit}} \"size\": \"{{size}}\", {{/limit}}  \"query\":{\"match\":{\"play_name\": \"{{play_name}}\"}}}{% endraw %}",
  "params": {
    "play_name": "Henry IV",
    "limit": true,
    "size": 2
  }
}
```

您也可以设计一个`if-else` 健康）状况。
此命令设置`size` 到`2` 如果`limit` 是`true`。否则，它设置了`size` 到`10`。

```json
GET _search/template
{
  "source": "{% raw %}{ {{#limit}} \"size\": \"2\", {{/limit}} {{^limit}} \"size\": \"10\", {{/limit}} \"query\":{\"match\":{\"play_name\": \"{{play_name}}\"}}}{% endraw %}",
  "params": {
    "play_name": "Henry IV",
    "limit": true
  }
}
```

### 循环

您也可以使用“截面标签”为每个循环实现一个：

```
{% raw %}{{#var}}{{.}}}{{/var}}{% endraw %}
```

什么时候`var` 是一个数组，搜索模板通过它迭代并创建一个`terms` 询问。

```json
GET _search/template
{
  "source": "{% raw %}{\"query\":{\"terms\":{\"play_name\":[\"{{#play_name}}\",\"{{.}}\",\"{{/play_name}}\"]}}}{% endraw %}",
  "params": {
    "play_name": [
      "Henry IV",
      "Othello"
    ]
  }
}
```

该模板被渲染为：

```json
GET _search/template
{
  "source": {
    "query": {
      "terms": {
        "play_name": [
          "Henry IV",
          "Othello"
        ]
      }
    }
  }
}
```

### 加入

您可以使用`join` 标签阵列的连接值（由逗号隔开）：

```json
GET _search/template
{
  "source": {
    "query": {
      "match": {
        "text_entry": "{% raw %}{{#join}}{{text_entry}}{{/join}}{% endraw %}"
      }
    }
  },
  "params": {
    "text_entry": [
      "To be",
      "or not to be"
    ]
  }
}
```

渲染为：

```json
GET _search/template
{
  "source": {
    "query": {
      "match": {
        "text_entry": "{0=To be, 1=or not to be}"
      }
    }
  }
}
```

### 转换为JSON

您可以使用`toJson` 标签将参数转换为其JSON表示：

```json
GET _search/template
{
  "source": "{\"query\":{\"bool\":{\"must\":[{\"terms\": {\"text_entries\": {% raw %}{{#toJson}}text_entries{{/toJson}}{% endraw %} }}] }}}",
  "params": {
    "text_entries": [
        { "term": { "text_entry" : "love" } },
        { "term": { "text_entry" : "soldier" } }
    ]
  }
}
```

渲染为：

```json
GET _search/template
{
  "source": {
    "query": {
      "bool": {
        "must": [
          {
            "terms": {
              "text_entries": [
                {
                  "term": {
                    "text_entry": "love"
                  }
                },
                {
                  "term": {
                    "text_entry": "soldier"
                  }
                }
              ]
            }
          }
        ]
      }
    }
  }
}
```

## 多个搜索模板

您可以使用多个搜索模板捆绑多个搜索模板，然后使用单个请求将其发送到您的OpenSearch集群中`msearch` 手术。
这节省了网络往返时间，因此与独立请求相比，您可以更快地获得响应。

```json
GET _msearch/template
{"index":"shakespeare"}
{"id":"if_search_template","params":{"play_name":"Henry IV","limit":false,"size":2}}
{"index":"shakespeare"}
{"id":"play_search_template","params":{"play_name":"Henry IV"}}
```

## 管理搜索模板

要列出所有脚本，请运行以下命令：

```json
GET _cluster/state/metadata?pretty&filter_path=**.stored_scripts
```

要检索特定的搜索模板，请运行以下命令：

```json
GET _scripts/<name_of_search_template>
```

要删除搜索模板，请运行以下命令：

```json
DELETE _scripts/<name_of_search_template>
```

---


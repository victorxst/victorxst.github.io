---
layout: default
title: IDs
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 30
---

# IDS查询

使用`ids` 查询以搜索具有一个或多个特定文档ID值的文档`_id` 场地。例如，以下查询请求具有IDS的文档`34229` 和`91296`：

```json
GET shakespeare/_search
{
  "query": {
    "ids": {
      "values": [
        34229,
        91296
      ]
    }
  }
}
```
{% include copy-curl.html %}

## 参数

查询接受以下参数。

范围| 数据类型| 描述
:--- | :--- | :---
`values` | 弦数| 要搜索的文档ID。必需的。


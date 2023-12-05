---
layout: default
title: 指数
parent: 工作负载参考
grand_parent: OpenSearch基准参考
nav_order: 65
redirect_from: /benchmark/workloads/indices/
---

# 指数

这`indices` 元素包含工作负载中使用的所有索引的列表。

## 例子

```json
"indices": [
    {
      "name": "geonames",
      "body": "geonames-index.json",
    }
]
```

## 配置选项

使用以下选项与`indices`：

范围| 必需的| 类型| 描述
:--- | :--- | :--- | :---
`name` | 是的| 细绳| 索引模板的名称。
`body` | 不| 细绳| 与Create Index API正文中使用的索引定义相对应的文件名。


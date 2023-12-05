---
layout: default
title: 故障排除
parent: SQL和PPL
nav_order: 88
redirect_from:
  - /search-plugins/sql/troubleshoot/
---

# 故障排除

SQL插件无状态，因此故障排除主要是针对特定查询失败的原因。

最常见的错误是可怕的空指针异常，该异常可能在解析错误期间或使用错误的HTTP方法（邮政与GET和VICE）时发生。POST方法和HTTP请求主体提供最一致的结果：

```json
POST _plugins/_sql
{
  "query": "SELECT * FROM my-index WHERE ['name.firstname']='saanvi' LIMIT 5"
}
```

如果查询没有您期望的行为，请使用`_explain` API查看翻译的查询，然后您可以对其进行故障排除。对于大多数操作，`_explain` 返回OpenSearch查询DSL。为了`UNION`，`MINUS`， 和`JOIN`，它返回类似于SQL执行计划的内容。

#### 示例请求

```json
POST _plugins/_sql/_explain
{
  "query": "SELECT * FROM my-index LIMIT  50"
}
```


#### 示例响应

```json
{
  "from": 0,
  "size": 50
}
```

## 索引映射验证例外

如果您看到以下验证异常，请确保查询中的索引不是索引模式，也没有多种类型：

```json
{
  "error": {
    "reason": "There was internal problem at backend",
    "details": "When using multiple indices, the mappings must be identical.",
    "type": "VerificationException"
  },
  "status": 503
}
```

如果这些步骤不起作用，请提交GitHub问题[这里](https://github.com/opensearch-project/sql/issues)。


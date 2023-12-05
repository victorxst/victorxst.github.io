---
layout: default
title: 辅助方法
parent: JavaScript客户端
nav_order: 2
---

# 辅助方法

辅助方法简化了复杂的API任务的使用。有关客户的完整API文档和其他示例，请参见[JS客户端API文档](https://opensearch-project.github.io/opensearch-js/2.2/index.html)。

## 散装助手

批量助手简化了提出复杂的散装API请求。

### 用法

以下代码创建了一个批量的助手实例：

```javascript
const { Client } = require('@opensearch-project/opensearch')
const documents = require('./docs.json')

const client = new Client({ ... })

const result = await client.helpers.bulk({
  datasource: documents,
  onDocument (doc) {
    return {
      index: { _index: 'example-index' }
    }
  }
})

console.log(result)
```
{% include copy.html %}

批量助手操作返回具有以下字段的对象：

```json
{
  total: number,
  failed: number,
  retry: number,
  successful: number,
  time: number,
  bytes: number,
  aborted: boolean
}
```

#### 批量辅助配置选项

创建新的散装助手实例时，您可以使用以下配置选项。

| 选项| 数据类型| 必需/默认| 描述
| :--- | :--- | :--- | :---
| `datasource` | 数组，异步发电机或可读的字符串或对象流| 必需的| 表示您需要创建，删除，索引或更新的文档。
| `onDocument` | 功能| 必需的| 在给定的每个文档中调用的函数`datasource`。它返回为此文档执行的操作。可选地，可以操纵文档`create` 和`index` 通过返回新文档作为函数结果的一部分操作。
| `concurrency` | 整数| 选修的。默认值为5。| 并行执行的请求数。
| `flushBytes` | 整数|  选修的。默认值为5,000,000。| 最大散装体大小以发送字节。
| `flushInterval` | 整数|  选修的。默认值为30,000。| 在阅读最后一份文档后，毫秒毫秒的时间才能等待尸体。
| `onDrop` | 功能| 选修的。默认为`noop`。| 每个文档要调用最大恢复次数后无法索引的每个文档的函数。
| `refreshOnCompletion` | 布尔| 选修的。默认值为false。| 是否应在批量操作结束时对所有受影响的索引进行刷新。
| `retries` | 整数|  选修的。默认为客户的`maxRetries` 价值。| 在操作之前进行操作的次数`onDrop` 被称为该文档。
| `wait` | 整数|  选修的。默认值为5,000。| 在重试操作之前等待毫秒的时间。

### 例子

以下示例说明了索引，创建，更新和删除散装助手操作。有关更多信息和高级索引操作，请参阅[`opensearch-js` 向导](https://github.com/opensearch-project/opensearch-js/tree/main/guides) 在github。

#### 指数

索引操作如果不存在，则会创建一个新文档，如果文档已经存在，则将重新创建文档。

以下批量操作将文档索引到`example-index`：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    return {
      index: { _index: 'example-index' }
    }
  }
})
```
{% include copy.html %}

以下批量操作将文档索引到`example-index` 使用文档覆盖：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    return [
      {
        index: { _index: 'example-index' }
      },
      { ...doc, createdAt: new Date().toISOString() }
    ]
  }
})
```
{% include copy.html %}

#### 创造

创建操作只有在尚不存在的文档时才创建一个新文档。

以下批量操作在`example-index`：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    return {
      create: { _index: 'example-index', _id: doc.id }
    }
  }
})
```
{% include copy.html %}

以下批量操作在`example-index` 使用文档覆盖：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    return [
      {
        create: { _index: 'example-index', _id: doc.id }
      },
      { ...doc, createdAt: new Date().toISOString() }
    ]
  }
})
```
{% include copy.html %}

#### 更新

更新操作更新文档，并发送了字段。该文档必须已经存在于索引中。

以下批量操作更新了文档`arrayOfDocuments`：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    // The update operation always requires a tuple to be returned, with the
    // first element being the action and the second being the update options.
    return [
      {
        update: { _index: 'example-index', _id: doc.id }
      },
      { doc_as_upsert: true }
    ]
  }
})
```
{% include copy.html %}

以下批量操作更新了文档`arrayOfDocuments` 使用文档覆盖：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    return [
      {
        update: { _index: 'example-index', _id: doc.id }
      },
      {
        doc: { ...doc, createdAt: new Date().toISOString() },
        doc_as_upsert: true
      }
    ]
  }
})
```
{% include copy.html %}

#### 删除

删除操作删除文档。

以下批量操作从`example-index`：

```javascript
client.helpers.bulk({
  datasource: arrayOfDocuments,
  onDocument (doc) {
    return {
      delete: { _index: 'example-index', _id: doc.id }
    }
  }
})
```
{％包括copy.html％


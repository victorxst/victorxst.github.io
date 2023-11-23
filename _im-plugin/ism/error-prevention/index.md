---
layout: default
title: ISM 错误预防
nav_order: 90
has_children: true
has_toc: false
redirect_from:
  - /im-plugin/ism/error-prevention/
  - /im-plugin/ism/error-prevention/index/
---

# ISM 错误预防

错误预防在执行索引状态管理（ISM）操作之前对其进行验证，以防止操作失败。它还在响应中输出来自操作验证结果的其他[索引解释 API]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/#explain-index)信息。以下各节列出了每个操作的验证规则和疑难解答。

---

#### 目录
1. 目录
{:toc}


---

## 过渡

在以下任何情况下，ISM 都不会对索引执行 `rollover` 操作：

- [索引不是写入索引]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/resolutions/#the-index-is-not-the-write-index).
- [索引没有别名]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/resolutions/#the-index-does-not-have-an-alias).
- [滚动更新策略不包含 rollover_alias 索引设置]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/resolutions/#the-rollover-policy-misses-rollover_alias-index-setting).
- [跳过滚动更新操作]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/resolutions/#skipping-rollover-action-is-true).
- [索引已成功使用别名进行滚动]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/resolutions/#this-index-has-already-been-rolled-over-successfully).

## 删除

在以下任何情况下，ISM 都不会对索引执行 `delete` 操作：

- 索引不存在。
- 索引名称无效。
- 索引是数据流的写入索引。

## force_merge

如果索引的数据集过大且超过阈值，则 ISM 不会对索引执行 `force_merge` 操作。

## replica_count

在以下任何情况下，ISM 都不会对索引执行 `replica_count` 操作：

- 数据量超过阈值。
- 分片数量超过最大值。

## 打开

在以下任何情况下，ISM 都不会对索引执行 `open` 操作：

- 索引被阻止。
- 分片数量超过最大值。

## read_only

在以下任何情况下，ISM 都不会对索引执行 `read_only` 操作：

- 索引被阻止。
- 数据量超过阈值。

## read_write

如果索引被阻止，ISM 不会对索引执行 `read_write` 操作。


## 关闭

在以下任何情况下，ISM 都不会对索引执行 `close` 操作：

- 索引不存在。
- 索引名称无效。

## index_priority

ISM 不会对没有 `read-only-allow-delete` 权限的索引执行 `index_priority` 操作。

## 快照

在以下任何情况下，ISM 都不会对索引执行 `snapshot` 操作：

- 索引不存在。
- 索引名称无效。

## 过渡

在以下任何情况下，ISM 都不会对索引执行 `transition` 操作：

- 索引不存在。
- 索引名称无效。
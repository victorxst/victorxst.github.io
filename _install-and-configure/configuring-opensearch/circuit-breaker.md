---
layout: default
title: 断路器设置
parent: 配置 OpenSearch
nav_order: 50
---

# 断路器设置

断路器可防止 OpenSearch 导致 Java OutOfMemoryError。父断路器指定所有子断路器的总可用内存量。子断路器指定自身的总可用内存量。

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 父断路器设置

OpenSearch 支持以下父断路器设置：

-  `indices.breaker.total.use_real_memory`（静态，布尔值）：如果 `true`，父断路器将考虑实际内存使用情况。否则，父断路器将考虑子断路器保留的内存量。缺省值为 `false`。

-  `indices.breaker.total.limit`（动态，百分比）：指定父断路器的初始内存限制。如果 `indices.breaker.total.use_real_memory` 是 `true`，则默认为 JVM 堆的 95%。如果 `indices.breaker.total.use_real_memory` 是 `false`，则默认为 JVM 堆的 70%。

## 现场数据断路器设置

字段数据断路器限制将字段加载到字段数据缓存中所需的堆内存。OpenSearch 支持以下字段数据断路器设置：

-  `indices.breaker.fielddata.limit`（动态，百分比）：指定现场数据断路器的内存限制。缺省值为 JVM 堆的 40%。

-  `indices.breaker.fielddata.overhead`（动态，双精度）：一个常量，将现场数据估计值乘以确定最终估计值。默认值为 1.03。

## 请求断路器设置

请求断路器限制了构建请求所需的数据结构所需的内存（例如，在计算聚合时）。OpenSearch 支持以下请求断路器设置：

-  `indices.breaker.request.limit`（动态，百分比）：指定请求断路器的内存限制。缺省值为 JVM 堆的 60%。

-  `indices.breaker.request.overhead`（动态，双精度）：一个常量，请求估计值乘以该常量以确定最终估计值。默认值为 1。

## 动态请求断路器设置

正在进行的请求断路器限制了传输和 HTTP 级别上当前正在运行的所有传入请求的内存使用量。请求的内存使用量基于请求的内容长度，包括原始请求所需的内存和表示请求的结构化对象。OpenSearch 支持以下动态请求断路器设置：

-  `network.breaker.inflight_requests.limit`（动态，百分比）：指定正在进行的请求断路器的内存限制。缺省值为 JVM 堆的 100%（因此，正在进行的请求的内存使用限制由父断路器的内存限制决定）。

-  `network.breaker.inflight_requests.overhead`（动态、双精度）：一个常量，通过该常数将正在进行的请求估计值相乘以确定最终估计值。默认值为 2。

## 脚本编译断路器设置

脚本编译断路器限制时间间隔内内联脚本编译的数量。OpenSearch 支持以下脚本编译断路器设置：

-  `script.max_compilations_rate`（动态、速率）：在给定上下文的时间间隔内编译的唯一动态脚本的最大数量。默认值为每 5 分钟 150 次（ `150/5m`）。

## 正则表达式断路器设置

正则表达式断路器启用或禁用正则表达式并限制其复杂性。OpenSearch 支持以下正则表达式断路器设置：

-  `script.painless.regex.enabled`（静态，字符串）：在 Painless 脚本中启用正则表达式。
    有效值为：
    -  `limited`：启用正则表达式并使用设置 `script.painless.regex.limit-factor` 限制其复杂性。
    -  `true`：启用正则表达式。关闭正则表达式断路器，并且不限制正则表达式的复杂性。
    -  `false`：禁用正则表达式。如果 Painless 脚本包含正则表达式，则返回错误。

    缺省值为 `limited`。

-  `script.painless.regex.limit-factor`（静态，整数）：仅当 `script.painless.regex.enabled` 设置为 `limited` 时才应用。限制 Painless 脚本中正则表达式的字符数。字符限制的计算方法是将脚本输入中的字符数乘以 `script.painless.regex.limit-factor`。默认值为 6（因此，如果输入有 5 个字符，则正则表达式中的最大字符数为 5 · 6 = 30）。

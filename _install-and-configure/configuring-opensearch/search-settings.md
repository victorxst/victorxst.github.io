---
layout: default
title: 搜索设置
parent: 配置 OpenSearch
nav_order: 80
---

# 搜索设置

OpenSearch 支持以下搜索设置：

-  `search.max_buckets`（动态，整数）：单个响应中允许的最大聚合存储桶数。缺省值为 `65535`。

-  `search.phase_took_enabled`（动态、布尔值）：允许在搜索响应中返回阶段级 `took` 时间值。缺省值为 `false`。

-  `search.allow_expensive_queries`（动态、布尔值）：允许或禁止成本高昂的查询。有关详细信息，请参阅[成本高昂的查询]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries)。

-  `search.default_allow_partial_results`（动态、布尔值）：一种集群级别的设置，允许在请求超时或分片失败时返回部分搜索结果。如果搜索请求包含参数 `allow_partial_search_results`，则该参数优先于此设置。缺省值为 `true`。

-  `search.cancel_after_time_interval`（动态，时间单位）：一种集群级别设置，用于指定搜索请求在分片级别取消之前可以运行的最长时间。达到此时间后，将停止请求并取消所有关联的任务。缺省值为 `-1`。

-  `search.default_search_timeout`（动态，时间单位）：一种集群级别设置，用于在协调节点级别设置所有搜索请求的默认超时。如果在搜索请求中指定了， `timeout` 则它优先于此设置。默认值为 `-1`（无超时）。

-  `search.default_keep_alive`（动态，时间单位）：指定滚动和时间点（PIT）搜索的默认保持活动值。由于请求可能会多次登陆分片（例如，在查询和提取阶段），因此 OpenSearch 会打开一个_请求上下文_在请求的整个持续时间内存在的分片，以确保每个分片请求的分片状态一致。在标准搜索中，一旦提取阶段完成，请求上下文就会关闭。对于滚动或 PIT 搜索，OpenSearch 会使请求上下文保持打开状态，直到显式关闭（或直到达到保持活动时间）。后台线程会定期检查所有打开的滚动和 PIT 上下文，并删除已超过其保持活动超时的上下文。该 `search.keep_alive_interval` 设置指定检查上下文是否过期的频率。该 `search.default_keep_alive` 设置是默认的到期截止时间。滚动或 PIT 请求可以显式指定保持活动状态，这优先于此设置。缺省值为 `5m`。

-  `search.keep_alive_interval`（静态，时间单位）：确定 OpenSearch 检查已超过其保持活动限制的请求上下文的时间间隔。缺省值为 `1m`。

-  `search.max_keep_alive`（动态，时间单位）：指定最大保持活动状态值。该 `max_keep_alive` 设置用作针对其他 `keep_alive` 设置（例如， `default_keep_alive`）和请求级保持活动设置（用于滚动和 PIT 上下文）的安全检查。如果请求超过任一情况下的值 `max_keep_alive`，则操作将失败。缺省值为 `24h`。

-  `search.low_level_cancellation`（动态，布尔值）：启用低级别请求取消。Lucene 的经典超时机制只在收集搜索结果时检查时间。但是，成本高昂的查询（如通配符或前缀）可能需要很长时间才能展开，然后才能开始收集结果。在这种情况下，查询可以运行一段时间，该时间段大于超时值。低级取消机制不仅在收集搜索结果时超时，而且在查询扩展阶段或执行任何 Lucene 操作之前超时，从而解决了这种情况。缺省值为 `true`。

-  `search.max_open_scroll_context`（动态，整数）：一个节点级设置，用于指定节点打开的滚动上下文的最大数量。缺省值为 `500`。

-  `search.request_stats_enabled`（动态、布尔值）：从协调器节点的角度打开相位计时统计信息的节点级收集。请求级统计信息跟踪搜索请求在每个不同搜索阶段中花费的时间（总计）。你可以使用检索这些计数器[节点统计 API]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-stats/)。缺省值为 `false`。

-  `search.highlight.term_vector_multi_value`（静态、布尔值）：指定突出显示多值字段值中的代码段。缺省值为 `true`。

## 时间点设置

有关 PIT 设置的信息，请参见[PIT 设置]({{site.url}}{{site.baseurl}}/search-plugins/point-in-time-api/#pit-settings)。

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

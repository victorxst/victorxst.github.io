---
layout: default
title: trace_peer_forwarder
parent: 处理器
grand_parent: 管道
nav_order: 115
---

# 跟踪对等转发器

这`trace_peer_forwarder` 处理器与[同行前锋]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/peer-forwarder/) 减少一半的事件数量[跟踪分析]({{site.url}}{{site.baseurl}}/data-prepper/common-use-cases/trace-analytics/) 管道。在跟踪分析中，每次事件通常都会重复`otel-trace-pipeline` 到`raw-pipeline` 和`service-map-pipeline`。当管道前进事件时，这会导致核心对等转发器发送多个HTTP请求对同一事件。您可以使用`trace peer forwarder` 一次通过`otel-trace-pipeline` 代替`raw-pipeline` 和`service-map-pipeline`，这阻止了不必要的HTTP请求。

您应该使用`trace_peer_forwarder` 对于有多个节点时，对于跟踪分析管道。

## 用法

开始`trace_peer_forwarder`，首先配置[同行前锋]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/peer-forwarder/)。然后创建一个`pipeline.yaml` 文件并指定`trace peer forwarder` 作为处理器。您可以配置`peer forwarder` 在你的`data-prepper-config.yaml` 文件。有关更多详细信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

请参阅以下示例`pipeline.yaml` 文件：

```yaml
otel-trace-pipeline:
  delay: "100"
  source:
    otel_trace_source:
  processor:
    - trace_peer_forwarder:
  sink:
    - pipeline:
        name: "raw-pipeline"
    - pipeline:
        name: "service-map-pipeline"
raw-pipeline:
  source:
    pipeline:
      name: "entry-pipeline"
  processor:
    - otel_trace_raw:
  sink:
    - opensearch:
service-map-pipeline:
  delay: "100"
  source:
    pipeline:
      name: "entry-pipeline"
  processor:
    - service_map_stateful:
  sink:
    - opensearch:
```

在上述`pipeline.yaml` 文件，事件在`otel-trace-pipeline` 到目标对等，并且没有进行转发`raw-pipeline` 或者`service-map-pipeline`。此过程有助于通过转发事件（作为HTTP请求）而不是两次来改善网络性能。


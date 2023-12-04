---
layout: default
title: 脚本公制
parent: 公制聚合
grand_parent: 聚合
nav_order: 100
redirect_from:
  - /query-dsl/aggregations/metric/scripted-metric/
---

# 脚本公制聚合

这`scripted_metric` 公制是多型-返回从指定脚本计算的度量标准的值指标聚合。

脚本有四个阶段：初始阶段，地图阶段，联合收割机阶段和减少阶段。

*`init_script`：（可选）设置初始状态并在任何文档集合之前执行。
*`map_script`：检查`type` 字段并在收集的文档上执行汇总。
*`combine_script`：汇总从每个碎片返回的状态。汇总值将返回到协调节点。
*`reduce_script`：提供对变量状态的访问；这个变量结合了`combine_script` 在每个碎片上陷入阵列。

以下示例将Web日志数据中的不同HTTP响应类型汇总：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggregations": {
    "responses.counts": {
      "scripted_metric": {
        "init_script": "state.responses = ['error':0L,'success':0L,'other':0L]",
        "map_script": """
              def code = doc['response.keyword'].value;
                 if (code.startsWith('5') || code.startsWith('4')) {
                  state.responses.error += 1 ;
                  } else if(code.startsWith('2')) {
                   state.responses.success += 1;
                  } else {
                  state.responses.other += 1;
                }
             """,
        "combine_script": "state.responses",
        "reduce_script": """
            def counts = ['error': 0L, 'success': 0L, 'other': 0L];
                for (responses in states) {
                 counts.error += responses['error'];
                  counts.success += responses['success'];
                counts.other += responses['other'];
        }
        return counts;
        """
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
...
"aggregations" : {
  "responses.counts" : {
    "value" : {
      "other" : 0,
      "success" : 12832,
      "error" : 1242
    }
  }
}
```


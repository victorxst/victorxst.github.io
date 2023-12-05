---
layout: default
title: 异常结果映射
parent: 异常检测
nav_order: 6
redirect_from: 
  - /monitoring-plugins/ad/result-mapping/
---

# 异常结果映射

如果启用了自定义结果索引，则异常检测插件将结果存储在您自己的索引中。

如果异常检测器未检测到异常，则结果具有以下格式：

```json
{
  "detector_id": "kzcZ43wBgEQAbjDnhzGF",
  "schema_version": 5,
  "data_start_time": 1635898161367,
  "data_end_time": 1635898221367,
  "feature_data": [
    {
      "feature_id": "processing_bytes_max",
      "feature_name": "processing bytes max",
      "data": 2322
    },
    {
      "feature_id": "processing_bytes_avg",
      "feature_name": "processing bytes avg",
      "data": 1718.6666666666667
    },
    {
      "feature_id": "processing_bytes_min",
      "feature_name": "processing bytes min",
      "data": 1375
    },
    {
      "feature_id": "processing_bytes_sum",
      "feature_name": "processing bytes sum",
      "data": 5156
    },
    {
      "feature_id": "processing_time_max",
      "feature_name": "processing time max",
      "data": 31198
    }
  ],
  "execution_start_time": 1635898231577,
  "execution_end_time": 1635898231622,
  "anomaly_score": 1.8124904404395776,
  "anomaly_grade": 0,
  "confidence": 0.9802940756605277,
  "entity": [
    {
      "name": "process_name",
      "value": "process_3"
    }
  ],
  "model_id": "kzcZ43wBgEQAbjDnhzGF_entity_process_3",
  "threshold": 1.2368549346675202
}
```

## 响应身体场

场地| 描述
:--- | :---
`detector_id` | 识别检测器的独特ID。
`schema_version` | 结果索引的映射版本。
`data_start_time` | 聚合数据的检测范围的开始。
`data_end_time` | 聚合数据的检测范围的末端。
`feature_data` | 汇总数据点的数组`data_start_time` 和`data_end_time`。
`execution_start_time` | 对于产生异常结果的特定运行，检测器的实际开始时间。此开始时间包括您可以将其设置为延迟数据收集的窗口延迟参数。窗口延迟是`execution_start_time` 和`data_start_time`。
`execution_end_time` | 对于产生异常结果的特定运行的检测器的实际结束时间。
`anomaly_score` | 表示异常的相对严重程度。分数越高，数据点的异常越多。
`anomaly_grade` | 标准化版本的`anomaly_score` 在0到1之间的比例。
`confidence` | 准确性的可能性`anomaly_score`。这个数字越接近1，精度越高。在运行检测器的试用期间，由于其暴露于有限的数据，置信度较低（<0.9）。
`entity` | 实体是特定类别字段值的组合。它包括类别字段的名称和值。在上一个示例中`process_name` 是类别字段，也是一个过程之一`process_3` 是该领域的价值。这`entity` 场仅出现高-基数检测器（您选择了类别字段）。
`model_id` | 标识模型的唯一ID。如果检测器是一个-流检测器（没有类别字段），它只有一个模型。如果检测器很高-基数检测器（具有一个或多个类别字段），它可能具有多个模型，一个用于每个实体。
`threshold` | 检测器将数据点分类为异常的标准之一是`anomaly_score` 必须超过动态阈值。该字段记录当前阈值。

如果异常检测器检测到异常，则结果具有以下格式：

```json
{
  "detector_id": "fylE53wBc9MCt6q12tKp",
  "schema_version": 0,
  "data_start_time": 1635927900000,
  "data_end_time": 1635927960000,
  "feature_data": [
    {
      "feature_id": "processing_bytes_max",
      "feature_name": "processing bytes max",
      "data": 2291
    },
    {
      "feature_id": "processing_bytes_avg",
      "feature_name": "processing bytes avg",
      "data": 1677.3333333333333
    },
    {
      "feature_id": "processing_bytes_min",
      "feature_name": "processing bytes min",
      "data": 1054
    },
    {
      "feature_id": "processing_bytes_sum",
      "feature_name": "processing bytes sum",
      "data": 5032
    },
    {
      "feature_id": "processing_time_max",
      "feature_name": "processing time max",
      "data": 11422
    }
  ],
  "anomaly_score": 1.1986675882872033,
  "anomaly_grade": 0.26806225550178464,
  "confidence": 0.9607519742565531,
  "entity": [
    {
      "name": "process_name",
      "value": "process_3"
    }
  ],
  "approx_anomaly_start_time": 1635927900000,
  "relevant_attribution": [
    {
      "feature_id": "processing_bytes_max",
      "data": 0.03628638020431366
    },
    {
      "feature_id": "processing_bytes_avg",
      "data": 0.03384479053991436
    },
    {
      "feature_id": "processing_bytes_min",
      "data": 0.058812549572819096
    },
    {
      "feature_id": "processing_bytes_sum",
      "data": 0.10154576265526988
    },
    {
      "feature_id": "processing_time_max",
      "data": 0.7695105170276828
    }
  ],
  "expected_values": [
    {
      "likelihood": 1,
      "value_list": [
        {
          "feature_id": "processing_bytes_max",
          "data": 2291
        },
        {
          "feature_id": "processing_bytes_avg",
          "data": 1677.3333333333333
        },
        {
          "feature_id": "processing_bytes_min",
          "data": 1054
        },
        {
          "feature_id": "processing_bytes_sum",
          "data": 6062
        },
        {
          "feature_id": "processing_time_max",
          "data": 23379
        }
      ]
    }
  ],
  "threshold": 1.0993584705913992,
  "execution_end_time": 1635898427895,
  "execution_start_time": 1635898427803
}
```

您可以看到以下其他字段：

场地| 描述
:--- | :---
`relevant_attribution` | 表示每个输入变量的贡献。归因的总和将其标准化为1。
`expected_values` | 每个功能的预期值。

有时，检测器可能迟到异常。
假设探测器看到三元组的随机组合{1，2，3}和{2，4，5}，与`slow weeks` 和`busy weeks`， 分别。例如1、2、3、1、2、3、2、4、5、1、2、3、2、4、5，等等。
如果检测器遇到模式{2、2，x}，并且尚未看到X，则检测器会渗透该模式是异常的，但是目前无法确定2个是原因。如果x = 3，则检测器知道这是未完成三倍的第一个2，如果x = 5，则是第二个。如果是第一个2，则检测器将检测器检测到较晚的异常。

如果检测器检测到较晚的异常，则结果具有以下其他字段：

场地| 描述
:--- | :---
`past_values` | 触发异常的实际输入。如果`past_values` 为null，归因或期望值来自当前输入。如果`past_values` 不是零，而是属性或期望值来自过去的输入（例如，数据的前两个步骤[1,2,3]）。
`approx_anomaly_start_time` | 触发异常的实际输入的大约时间。该字段可帮助您了解何时检测器标记异常。两者都单-流和高-基数检测器不会查询以前的异常结果，因为这些查询是昂贵的操作。高昂的成本特别高-可能具有很多实体的基数检测器。如果数据不是连续的，则该领域的准确性很低，并且检测器检测到异常的实际时间可以更早。

```json
{
  "detector_id": "kzcZ43wBgEQAbjDnhzGF",
  "confidence": 0.9746820962328963,
  "relevant_attribution": [
    {
      "feature_id": "deny_max1",
      "data": 0.07339452532666227
    },
    {
      "feature_id": "deny_avg",
      "data": 0.04934972719948845
    },
    {
      "feature_id": "deny_min",
      "data": 0.01803003656061806
    },
    {
      "feature_id": "deny_sum",
      "data": 0.14804918212089874
    },
    {
      "feature_id": "accept_max5",
      "data": 0.7111765287923325
    }
  ],
  "task_id": "9Dck43wBgEQAbjDn4zEe",
  "threshold": 1,
  "model_id": "kzcZ43wBgEQAbjDnhzGF_entity_app_0",
  "schema_version": 5,
  "anomaly_score": 1.141419389056506,
  "execution_start_time": 1635898427803,
  "past_values": [
    {
      "feature_id": "processing_bytes_max",
      "data": 905
    },
    {
      "feature_id": "processing_bytes_avg",
      "data": 479
    },
    {
      "feature_id": "processing_bytes_min",
      "data": 128
    },
    {
      "feature_id": "processing_bytes_sum",
      "data": 1437
    },
    {
      "feature_id": "processing_time_max",
      "data": 8440
    }
  ],
  "data_end_time": 1635883920000,
  "data_start_time": 1635883860000,
  "feature_data": [
    {
      "feature_id": "processing_bytes_max",
      "feature_name": "processing bytes max",
      "data": 1360
    },
    {
      "feature_id": "processing_bytes_avg",
      "feature_name": "processing bytes avg",
      "data": 990
    },
    {
      "feature_id": "processing_bytes_min",
      "feature_name": "processing bytes min",
      "data": 608
    },
    {
      "feature_id": "processing_bytes_sum",
      "feature_name": "processing bytes sum",
      "data": 2970
    },
    {
      "feature_id": "processing_time_max",
      "feature_name": "processing time max",
      "data": 9670
    }
  ],
  "expected_values": [
    {
      "likelihood": 1,
      "value_list": [
        {
          "feature_id": "processing_bytes_max",
          "data": 905
        },
        {
          "feature_id": "processing_bytes_avg",
          "data": 479
        },
        {
          "feature_id": "processing_bytes_min",
          "data": 128
        },
        {
          "feature_id": "processing_bytes_sum",
          "data": 4847
        },
        {
          "feature_id": "processing_time_max",
          "data": 15713
        }
      ]
    }
  ],
  "execution_end_time": 1635898427895,
  "anomaly_grade": 0.5514172746375128,
  "entity": [
    {
      "name": "process_name",
      "value": "process_3"
    }
  ],
  "approx_anomaly_start_time": 1635883620000
}
```


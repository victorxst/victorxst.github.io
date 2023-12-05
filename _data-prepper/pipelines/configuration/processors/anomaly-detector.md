---
layout: default
title: anomaly_detector
parent: 处理器
grand_parent: 管道
nav_order: 45
---

# Anomaly_detector

异常检测器处理器采用结构化数据，并在可以在该数据中配置的字段上运行异常检测算法。数据必须是用于检测异常的异常检测算法的整数或实际数字。在异常检测器处理器之前，将聚合处理器部署在管道中可以帮助您获得最佳结果，因为聚合处理器会通过键自动汇总事件并将其保留在同一主机上。例如，如果您正在从特定的IP地址中搜索潜伏期的异常，并且如果所有事件都转到同一主机，则主机将对这些事件有更多数据。此附加数据可更好地培训机器学习（ML）算法，从而可以更好地检测异常。

## 配置

您可以通过指定键和选定模式的选项来配置异常检测器处理器。您可以使用以下选项来配置异常检测器处理器。

| 姓名| 必需的| 描述|
| ：--- | ：--- | ：--- |
| `keys` | 是的| 非-订购`List<String>` 这用作ML算法的输入，以检测列表中键值中的异常。至少需要一个钥匙。
| `mode` | 是的| 用于检测异常的ML算法（或模型）。您必须提供模式。看[Random_cut_forest模式](#random_cut_forest-mode)。
| `identification_keys` | 不| 如果提供，将在此密钥的每个唯一实例中检测到异常。例如，如果您提供`ip` 字段，将针对每个唯一的IP地址分别检测到异常。
| `cardinality_limit` | 不| 如果使用`identification_keys` 设置，将为各种基数创建一个新的ML模型。这可能会导致大量的内存使用量，因此设置型号数量很有帮助。默认限制为5000。
| `verbose` | 不| RCF将尝试自动学习并减少检测到的异常数量。例如，如果延迟始终在50到100之间，然后突然跳至1000左右，则仅检测到过渡后的第一个或两个数据点（除非有其他尖峰/异常）。同样，对于重复的尖峰至相同的级别，RCF可能会在几个初始的尖峰上消除许多尖峰。这是因为默认设置是最大程度地减少检测到的警报数量。设置`verbose` 设置为`true` 将导致RCF始终检测到这些重复的病例，这对于检测持续延长时间的异常行为可能是有用的。


### 钥匙

输入事件中存在异常检测器处理器中使用的键。例如，如果输入事件是`{"key1":value1, "key2":value2, "key3":value3}`，然后任何键（例如`key1`，，，，`key2`，，，，`key3`）在该输入事件中，只要其值，就可以用作异常检测器键（例如`value1`，，，，`value2`，，，，`value3`）是整数或实际数字。

### Random_cut_forest模式

随机切割森林（RCF）ML算法是一种无监督的算法，用于检测数据集中的异常数据点。为了检测异常，异常检测器处理器使用`random_cut_forest` 模式。

| 姓名| 描述|
| ：--- | ：--- |
| `random_cut_forest` | 使用RCF ML算法处理事件以检测异常。| 

RCF是一种无监督的ML算法，用于检测数据集中的异常数据点。Data Prepper使用RCF通过将配置密钥的值传递给RCF来检测数据中的异常情况。例如，当发送延迟值为11.5的事件时，生成以下异常事件：


 ```JSON
  {"latency"：11.5，"deviation_from_expected"：[10.469302736820003]，"grade"：1.0}
```

In this example, `deviation_from_expected` is a list of deviations for each of the keys from their corresponding expected values, and `grade` is the anomaly grade that indicates the anomaly severity.
     

You can configure `random_cut_forest` mode with the following options. 

| Name | Default value | Range | Description |
| :--- | :--- | :--- | :--- |
| `shingle_size` | `4` | 1--60 | The shingle size used in the ML algorithm. |
| `sample_size` | `256` | 100--2500 | The sample size used in the ML algorithm. |
| `time_decay` | `0.1` | 0--1.0 | The time decay value used in the ML algorithm. Used as the mathematical expression `timeDecay` divided by `SampleSize` in the ML algorithm. |
| `type` | `metrics` | N/A | The type of data sent to the algorithm. |
| `version` | `1.0` | N/A | The algorithm version number. |

## Usage

To get started, create the following `pipeline.yaml` file. You can use the following pipeline configuration to look for anomalies in the `latency` field in events that are passed to the processor. Then you can use the following YAML configuration file `random_cut_forest` mode to detect anomalies:

```yaml
广告-管道：
  来源：
    ...
  ...
  处理器：
    - Anomaly_detector：
        钥匙：[["latency"这是给出的
        模式：
            Random_cut_forest：
```

When you run the anomaly detector processor, the processor extracts the value for the `latency` key, and then passes the value through the RCF ML algorithm. You can configure any key that comprises integers or real numbers as values. In the following example, you can configure `bytes` or `latency` as the key for an anomaly detector. 

`{"ip":"1.2.3.4", "bytes":234234, "latency":0.2}`


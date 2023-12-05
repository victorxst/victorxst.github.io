---
layout: default
title: 根本原因分析
nav_order: 50
parent: 性能分析仪
has_children: true
redirect_from:
  - /monitoring-plugins/pa/rca/index/
---

# 根本原因分析

OpenSearch Performance Analyzer插件（PA）捕获了OpenSearch和JVM活动，以及其较低-级别资源使用率（例如磁盘，网络，CPU和内存）。基于此仪器，性能分析仪计算和公开诊断指标，以便管理员可以测量和理解其OpenSearch集群中的瓶颈。

根本原因分析框架（RCA）使用来自PA的信息来提醒管理员有关其群集可能正在遇到的性能和可用性问题的根本原因。

在广泛的笔触中，该框架可帮助您访问OpenSearch节点运行性能分析仪的数据流。您可以编写Java的片段，以选择对您重要的流并评估流的PA指标针对某些阈值。随着RCA运行，您可以使用REST API访问每个分析的状态。

要了解有关根本原因分析的更多信息，请参阅[它的存储库在github上](https://github.com/opensearch-project/performance-analyzer-rca)。


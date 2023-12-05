---
layout: default
title: OpenSearch集成
nav_order: 1
has_children: false
nav_exclude: true
---

# OpenSearch集成
引入2.9
{: .label .label-purple }

OpenSearch Integrations是一个简单的起点，OpenSearch和OpenSearch仪表板用户可以用来可视化和了解特定资源（例如Nginx）的日志和度量数据。 _ Integration_包含一束元数据，数据映射和可视化，使从集成资源监视数据变得易于易于监视。

OpenSearch项目向您寻求有关此功能的反馈。让我们知道[OpenSearch论坛](https://forum.opensearch.org/) OpenSearch集成对您有效或如何改进。
{： 。标签-黄色的}

## 设置OpenSearch集成

有关最新开发人员信息，包括示例代码，文章，教程和API参考，请参见以下资源：

- [集成存储库](https://github.com/opensearch-project/observability/tree/e18cf354fd7720a6d5df6a6de5d53e51a9d43127/integrations) 在github上
- [集成创建指南](https://github.com/opensearch-project/dashboards-observability/wiki/Integration-Creation-Guide)
- [集成文档参考](https://github.com/opensearch-project/dashboards-observability/wiki/Integration-Documentation-Reference)
- [OpenSearch仪表板的可观察性插件](https://github.com/opensearch-project/dashboards-observability/wiki)
 
## 集成模式

OpenSearch集成模式概述了如何捕获，分析和可视化数据。它包括监视工具的选择和配置，数据收集方法，数据存储和保留策略以及可视化和警报机制。它遵循[OpentElemetry协议约定](https://github.com/open-telemetry)，使用OpenSearch[简单的模式可观察到](https://opensearch.org/docs/latest/observing-your-data/ssfo/) 处理从OpenTelemetry（Otel）模式到物理索引映射模板的翻译。

在[OpenSearch可观察性read.me文件](https://github.com/opensearch-project/opensearch-catalog/blob/main/docs/schema/observability/README.md) 在[OpenSearch可观察性Wiki](https://github.com/opensearch-project/dashboards-observability/wiki/OpenSearch-Observability--Home#observability-schema)。

## 开始

使用OpenSearch仪表板接口，您可以连接数据，应用程序和进程，以便您可以集中管理所使用的服务。所有集成都可以在单个视图中使用，并且OpenSearch仪表板从主页和主菜单从那里引导您那里。

了解如何使用OpenSearch仪表板接口进行以下操作：

- 访问集成
- 查看集成
- 添加集成

以下图像为您提供了集成接口的快照：

![开始进行集成演示]({{site.url}}{{site.baseurl}}/images/integrations/nginx-integration.gif)

### 访问集成

要访问集成，请打开OpenSearch仪表板并选择**集成** 来自**管理** 菜单。接口显示已安装和可用集成。

## 查看集成

要查看集成，请查看与集成关联的仪表板。如果集成没有关联的仪表板，请选择下列出的所需集成**安装** 窗户。查看集成细节，例如资产和字段。

## 添加集成

如果您没有安装任何集成，则会提示您从集成接口安装它们。支持的集成在**可用的** 窗户。

要添加集成，请选择所需的预包装资产。当前，OpenSearch集成有两个流：添加或尝试。以下示例使用尝试流动：

1. 在**集成** 页面，选择**nginx仪表板**。
2. 选择**尝试一下** 按钮。 _尝试自动创建样本索引模板，将示例数据添加到模板中，然后基于该数据创建集成。_
3. 查看资产列表，然后选择仪表板资产。
4. 预览数据可视化和示例数据详细信息。



---
layout: default
title: 扩展
nav_order: 10
---

# 扩展

扩展是一个实验特征。因此，我们不建议在生产环境中使用扩展。有关扩展进度的更新，或者如果您想离开反馈可以帮助改进功能，请参阅[Github上的问题](https://github.com/opensearch-project/OpenSearch/issues/2447)。
{： 。警告}

在引入扩展之前，插件是扩展OpenSearch功能的唯一方法。但是，插件存在很大的缺点：它们需要频繁的更新才能与OpenSearch Core保持最新状态，它们构成安全风险，因为它们在与OpenSearch相同的过程中运行，并且更新或安装它们需要完整的群集重新启动。此外，如果失败，插件可能会致命地影响集群。

扩展提供了一种更简单，更安全的方法来自定义OpenSearch。扩展支持所有插件功能，并让您为OpenSearch构建其他模块化功能。这[Java的OpenSearch SDK](https://github.com/opensearch-project/opensearch-sdk-java/) 提供可以用来开发扩展名的类和接口库。扩展名是从OpenSearch Core分离的，不需要频繁的更新。此外，它们可以在单独的过程或另一个节点上运行，并且可以在群集运行时安装。

## 入门

使用以下文档开始扩展：

### 步骤1：学习基础知识

阅读[设计文档](https://opensearch-project.github.io/opensearch-sdk-java/DESIGN.html) 了解扩展体系结构以及扩展的工作原理。

### 步骤2：尝试一下

尝试通过以下详细步骤在[开发者指南的入门部分](https://opensearch-project.github.io/opensearch-sdk-java/DEVELOPER_GUIDE.html#getting-started)。

### 步骤3：创建自己的扩展

通过按照说明中的说明来开发自定义创建，读，更新，删除（crud）扩展[本教程](https://opensearch-project.github.io/opensearch-sdk-java/CREATE_YOUR_FIRST_EXTENSION.html)。

### 步骤4：学习如何部署扩展

有关构建，测试和运行扩展的说明，请参阅[在开发人员指南中开发自己的扩展部分](https://opensearch-project.github.io/opensearch-sdk-java/DEVELOPER_GUIDE.html#developing-your-own-extension)。

<！-- TODO：发行后添加链接
## 扩展Javadoc

有关完整的扩展类别层次结构，请参阅[Javadoc](Link TBD)。
-->

## 插件迁移

这[异常检测插件](https://github.com/opensearch-project/anomaly-detection) 就是现在[实施为扩展](https://github.com/opensearch-project/anomaly-detection/tree/feature/extensions)。有关详细信息，请参阅[这个Github问题](https://github.com/opensearch-project/OpenSearch/issues/3635)。

有关将现有插件迁移到扩展名的提示，请参阅[插件迁移文档](https://opensearch-project.github.io/opensearch-sdk-java/PLUGIN_MIGRATION.html)


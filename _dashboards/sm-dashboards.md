---
layout: default
title: 快照管理
nav_order: 90
redirect_from:
  - /dashboards/admin-ui-index/sm-dashboards/
---

# 快照管理

[快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/index/) 是集群索引和状态的备份。该状态包括集群设置，节点信息，索引元数据（映射，设置，模板）和分配分配。OpenSearch仪表板中的快照管理（SM）接口为拍摄和恢复快照提供了统一的解决方案。

接口的示例如下图所示。

![快照管理用户界面]({{site.url}}{{site.baseurl}}/images/dashboards/snapshots-UI.png)

## 快照用例

快照有两个主要用途：

1. 从失败中恢复

    例如，如果簇健康变红，则可以从快照中恢复红色索引。

2. 从一个集群迁移到另一个集群

    例如，如果您要从概念验证到生产群集，则可以使用前者的快照并在后者上恢复它。

## 创建一个存储库

在创建SM策略之前，请设置用于快照的存储库。

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
2. 在左面板下，**快照管理**， 选择**存储库**。
3. 选择**创建存储库** 按钮。
4. 输入存储库名称，类型和位置。
5. （可选）选择**高级设置** 并输入此存储库作为JSON对象的其他设置。
#### 例子
```json
    {
        "chunk_size": null,
        "compress": false,
        "max_restore_bytes_per_sec": "40m",
        "max_snapshot_bytes_per_sec": "40m",
        "readonly": false
    }
```
6. 选择**添加** 按钮。

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/star-icon.png" class ="inline-icon" alt ="star icon"/> {:/}**笔记：** 如果您需要自动化快照创建，则可以使用快照策略。
{：.. note purple}

## 删除存储库

要删除快照存储库配置，请从**存储库** 列表，然后选择**删除** 按钮。

## 创建SM政策

创建一个SM策略来设置自动快照。SM策略定义了自动快照创建时间表和可选的自动删除时间表。

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
1. 在左面板下，**快照管理**， 选择**快照政策**。
1. 选择**创建策略** 按钮。
1. 在里面**政策设置** 部分：
    1.输入策略名称。
    1.（可选）输入策略说明。
1. 在里面**来源和目的地** 部分：
    1.选择或输入源索引作为列表或索引模式。
    1.选择用于快照的存储库。到[创建一个新的存储库](#creating-a-repository)，选择**创造** 按钮。
1. 在里面**快照时间表** 部分：
    1.选择所需的快照频率或输入自定义CRON表达式以获取快照频率。
    1.选择开始时间和时区。
1. 在里面**保留期** 部分：
    1.选择保留所有快照或指定保留条件（保留快照的最大年龄）。
    1.（可选）**其他设置**，选择保留快照，删除频率和删除启动时间的最小和最大数量。
1. 在里面**通知** 部分，选择要通知的快照活动。
1. （可选）**高级设置** 部分，选择所需的选项：
    - **在快照中包括群集状态**
    - **忽略不可用的索引**
    - **允许部分快照**
1. 选择**创造** 按钮。

## 查看，编辑或删除SM策略

您可以在策略详细信息页面上查看，编辑或删除SM策略。

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
1. 在左面板下，**快照管理**， 选择**快照政策**。
1. 单击**策略名称** 您要查看，编辑或删除的策略。<br>
策略详细信息页面显示了策略设置，快照时间表，快照保留期，通知以及最后的创建和删除。<br>如果快照创建或删除失败，您可以查看有关失败的信息**最后创建/删除** 部分。要查看故障消息，请单击**原因** 在里面**信息** 柱子。
1. 要编辑或删除SM策略，请选择**编辑** 或者**删除** 按钮。

## 启用，禁用或删除SM政策

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
1. 在左面板下，**快照管理**， 选择**快照政策**。
1. 在列表中选择一个或多个策略。
1. 要启用或禁用选定的SM策略，请选择**使能够** 或者**禁用** 按钮。删除选定的SM策略，**动作** 列表，选择**删除** 选项。

## 查看快照

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
1. 在左面板下，**快照管理**， 选择**快照**。
所有自动或手动拍摄的快照都显示在列表中。
1. 要查看快照，请单击其**姓名**。

## 拍摄快照

按照以下步骤手动进行快照：

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
1. 在左面板下，**快照管理**， 选择**快照**。
1. 选择**拍摄快照** 按钮。
1. 输入快照名称。
1. 选择或输入源索引作为列表或索引模式。
1. 选择快照的存储库。
1. （可选）**高级选项** 部分，选择所需的选项：
    - **在快照中包括群集状态**
    - **忽略不可用的索引**
    - **允许部分快照**
1. 选择**添加** 按钮。

## 删除快照

这**删除** 按钮[删除]({{site.url}}{{site.baseurl}}/api-reference/snapshots/delete-snapshot/) 存储库的快照。

1. 要查看您的存储库列表，请选择**存储库** 在下面**快照管理** 部分。
2. 要查看您的快照列表，请选择**快照** 在下面**快照管理** 部分。

## 恢复快照

1. 在OpenSearch仪表板主菜单中，选择**管理** >**快照管理**。
1. 在左面板下，**快照管理**， 选择**快照**。这**快照** 默认情况下选择选项卡。
1. 选择要还原的快照旁边的复选框。一个示例如下图：
    <img src ="{{site.url}}{{site.baseurl}}/images/restore-snapshot/restore-snapshot-main.png" alt ="Snapshots"> {：.img-体液}

    {::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/star-icon.png" class ="inline-icon" alt ="star icon"/> {:/}**笔记：** 您只能以状态恢复快照`Success` 或者`Partial`。快照的状态显示在**快照状态** 柱子。
    {：.. note purple}
1. 在里面**还原快照** 飞翔，选择用于恢复快照的选项。

    这**还原快照** Flyout列出了快照名称和状态。要查看快照中的索引列表，请选择下面的编号**指数** （例如，`27` 在下面的图像中）。该数字表示快照中的索引数。

    <img src ="{{site.url}}{{site.baseurl}}/images/restore-snapshot/restore-snapshot.png" alt ="Restore Snapshot" width="450">

    有关有关选项的更多信息**还原快照** 飞翔，看[还原快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#restore-snapshots)。

    **忽略丢失的索引**

    如果指定要从快照还原的索引，然后选择**忽略不可用的索引** 选项，还原操作忽略了快照中缺少的索引。例如，如果要还原`log1` 和`log2` 索引，但是`log2` 不在快照中，`log1` 已恢复和`log2` 被忽略。如果您不选择**忽略不可用的索引**，如果快照缺少要恢复的索引，则整个还原操作会失败。

    **自定义索引设置**

    您可以选择从快照恢复的索引来自定义一些设置：<br>
        ＆emsp;＆＆#x2022;选择**自定义索引设置** 复选框为指定的索引设置提供新值。所有新恢复的索引都将使用这些值，而不是快照中的值。<br>
        ＆emsp;＆＆#x2022;选择**忽略索引设置** 复选框以指定快照中的设置以忽略。所有新恢复的索引都将用于这些设置的群集默认值。

    下图集中的示例`index.number_of_replicas` 到`0`，`index.auto_expand_replicas` 到`true`， 和`index.refresh_interval` 和`index.max_script_fields` 对于所有新还原索引的群集默认值。

    <img src ="{{site.url}}{{site.baseurl}}/images/restore-snapshot/restore-snapshot-custom.png" alt ="Custom settings" width="450">

    有关索引设置的更多信息，请参阅[索引设置]({{site.url}}{{site.baseurl}}/im-plugin/index-settings/)。

    有关您无法更改或忽略的设置列表，请参见[还原快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#restore-snapshots)。

    选择选项后，选择**还原快照** 按钮。
1. （可选）要监视还原进度，请选择**查看还原活动** 在确认对话框中。您还可以通过选择**恢复正在进行的活动** 标签，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/restore-snapshot/restore-snapshot-activities.png" alt ="Restore Activities"> {：.img-体液}

    您可以查看已完成的工作百分比**地位** 柱子。快照还原完成后，**地位** 更改为`Completed (100%)`。

    {::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/star-icon.png" class ="inline-icon" alt ="star icon"/> {:/}**笔记：** 这**恢复正在进行的活动** 面板不持续。它仅显示当前还原操作的进度。如果多个还原操作正在运行，则面板将显示最新操作。
    {：.. note purple}
    要查看要恢复的每个索引的状态，请在**索引正在恢复** 列（在前面的图像中，`27 Indices` 关联）。这**索引正在恢复** 飞行（如下图所示）显示每个索引及其还原状态。

    <img src ="{{site.url}}{{site.baseurl}}/images/restore-snapshot/restore-snapshot-indices.png" alt ="Restore Indices"> {：.img-体液}

 还原操作完成后，在**指数** 控制板。在左图中查看索引**索引管理**， 选择**指数**。

<img src="{{site.url}}{{site.baseurl}}/images/restore-snapshot/restore-snapshot-indices-panel.png" alt="View Indices">{: .img-fluid}


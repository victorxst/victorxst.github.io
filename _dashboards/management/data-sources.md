---
layout: default
title: 数据源
nav_order: 110
has_children: true
---

# 数据源

OpenSearch数据源是OpenSearch可以连接到并摄入数据的应用程序。一旦您的数据源连接并摄入了数据，就可以使用索引，搜索和分析。[REST API]({{site.url}}{{site.baseurl}}/api-reference/index/) 或OpenSearch仪表板UI。

该文档重点是使用OpenSeach仪表板接口来连接和管理您的数据源。有关使用API连接数据源的信息，请参见链接的开发人员资源[下一步](#next-steps)。

## 先决条件

将数据源连接到OpenSearch的第一步是在系统上安装OpenSearch和OpenSearch仪表板。您可以按照[OpenSearch文档]({{site.url}}{{site.baseurl}}/install-and-configure/index/) 安装这些工具。

安装了OpenSearch和OpenSearch仪表板后，您可以使用仪表板将数据源连接到OpenSearch，然后使用仪表板来管理数据源，根据这些数据源创建索引模式，对特定数据源运行查询，并在特定的数据源中运行查询，然后在组合可视化的数据。一个仪表板。

配置[YAML文件]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/#configuration-file) 和安装`dashboards-observability` 和`opensearch-sql` 插件是必要的。有关更多信息，请参阅[OpenSearch插件]({{site.url}}{{site.baseurl}}/install-and-configure/plugins/)。

## 创建数据源连接

数据源连接指定连接到数据源所需的参数。这些参数形成了数据源的连接字符串。使用仪表板，您可以添加新的数据源连接或管理现有的连接。

以下步骤指导您介绍创建数据源连接的基础知识：

1. 在OpenSearch仪表板主菜单中，选择**仪表板管理** >**数据源** >**创建数据源连接**。UI如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/data-source-UI.png" alt ="Connecting a data source UI" 宽度="700"/>

2. 通过将适当的信息输入到该数据源连接**连接详细信息** 和**身份验证方法** 字段。
   
   - 在下面**连接详细信息**，输入标题和端点URL。对于本教程，使用URL`http://localhost:5601/app/management/opensearch-dashboards/dataSources`。输入描述是可选的。

   - 在下面**身份验证方法**，从下拉列表中选择一个身份验证方法。选择身份验证方法后，将出现该方法的适用字段。然后，您可以输入所需的详细信息。身份验证方法选项是：
       - **没有身份验证**：没有使用身份验证来连接到数据源。
       - **用户名密码**：基本的用户名和密码用于连接到数据源。
       - **AWS SIGV4**：AWS签名版本4认证请求用于连接到数据源。AWS签名版本4需要一个访问密钥和秘密键。
         - 对于AWS签名版本4身份验证，首先指定**地区**。接下来，在**服务名称** 列表。选项是**Amazon OpenSearch服务** 和**Amazon OpenSearch无服务器**。最后，输入**访问密钥** 和**密钥** 用于授权。

    填充所需字段后，**测试连接** 和**创建数据源** 按钮变得活跃。您可以选择**测试连接** 确认连接有效。

3. 选择**创建数据源** 保存您的设置。连接是创建的。活动窗口返回到**数据源** 主页和新连接出现在数据源列表中。

4. 要删除数据源连接，请选择数据源左侧的复选框**标题** 然后选择**删除1连接** 按钮。支持为多个连接选择多个复选框。下图显示了一个示例UI。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/delete-data-source.png" alt ="Deleting a data source UI" 宽度="700"/>

### 修改数据源连接

要更改数据源连接，请在列表中选择一个连接**数据源** 主页。这**连接详细信息** 窗口打开。

进行更改**连接详细信息**，编辑一个或两个**标题** 和**描述** 字段和选择**保存更改** 在较低-屏幕的右角。您也可以在此处取消更改。更改**身份验证方法**，选择其他身份验证方法，输入您的凭据（如果适用），然后选择**保存更改** 在较低-屏幕的右角。更改已保存。

什么时候**用户名密码** 是选定的身份验证方法，您可以通过选择来更新密码**更新存储的密码** 旁边**密码** 场地。在流行音乐中-向上窗口，在第一个字段中输入新密码，然后再次在第二个字段中输入以确认。选择**更新存储的密码** 在流行音乐中-向上窗口。新密码已保存。选择**测试连接** 确认连接有效。

什么时候**AWS SIGV4** 是所选的身份验证方法，您可以通过选择来更新凭据**更新存储的AWS凭据**。在流行音乐中-向上窗口，在第一个字段中输入新的访问密钥，并在第二字段中输入一个新的秘密密钥。选择**更新存储的AWS凭据** 在流行音乐中-向上窗口。新的凭据被保存。选择**测试连接** 在上部-屏幕的右角确认连接有效。

要删除数据源连接，请选择“删除图标”（{:: nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/trash-can-icon.png" class ="inline-icon" alt ="delete icon"/> {：/}）。

## 创建索引模式

创建数据源连接后，您可以为数据源创建索引模式。_index模式_是一个模板，OpenSearch用来从数据源创建数据索引。看[索引模式]({{site.url}}{{site.baseurl}}/dashboards/management/index-patterns/) 有关更多信息和教程。

## 下一步

- 学习关于[管理索引模式]({{site.url}}{{site.baseurl}}/dashboards/management/index-patterns/) 通过OpenSearch仪表板。
- 学习关于[使用索引管理索引数据]({{site.url}}{{site.baseurl}}/dashboards/im-dashboards/index/) 通过OpenSearch仪表板。
- 了解如何连接[多个数据源]({{site.url}}{{site.baseurl}}/dashboards/management/multi-data-sources/)。
- 了解如何[通过OpenSearch仪表板连接OpenSearch和Amazon S3]({{site.url}}{{site.baseurl}}/dashboards/management/S3-data-source/)。
- 了解[集成]({{site.url}}{{site.baseurl}}/integrations/index/) 工具，它使您可以灵活地使用各种数据摄入方法并连接仪表板UI的数据。



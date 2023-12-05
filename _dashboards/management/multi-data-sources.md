---
layout: default
title: 配置和使用多个数据源
parent: 数据源
nav_order: 10
redirect_from: 
  - /dashboards/discover/multi-data-sources/
---

# 配置和使用多个数据源

您可以在OpenSearch仪表板中摄入，处理和分析来自多个数据源的数据。您在**仪表板管理** >**数据源** 应用，如下图所示。


<img src="{{site.url}}{{site.baseurl}}/images/dashboards/data-sources-management.png" alt="Dashboards Management Data sources main screen" width="700">

## 入门

以下教程指导您配置和使用多个数据源。

### 步骤1：修改YAML文件设置

要使用多个数据源，您必须启用`data_source.enabled` 环境。默认情况下它是禁用的。启用多个数据源：

1. 打开OpenSearch仪表板配置文件的本地副本，`opensearch_dashboards.yml`。如果您没有副本，[`opensearch_dashboards.yml`](https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/config/opensearch_dashboards.yml) 可在Github上找到。
2. 放`data_source.enabled:` 到`true` 并保存yaml文件。
3. 重新启动OpenSearch仪表板容器。
4. 通过连接到OpenSearch仪表板并查看该配置设置是否正确配置了配置设置**仪表板管理** 导航菜单。**数据源** 出现在侧边栏中。您会看到类似于下图的视图。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/multidatasources.png" alt ="Data sources in sidebar within Dashboards Management" 宽度="700">

### 步骤2：创建一个新的数据源连接

数据源连接指定连接到数据源所需的参数。这些参数形成了数据源的连接字符串。

创建一个新的数据源连接：

1. 在OpenSearch仪表板主菜单中，选择**仪表板管理** >**数据源** >**创建数据源连接**。
2. 将所需信息添加到每个字段以配置**连接详细信息** 和**身份验证方法**。
   
    - 在下面**连接详细信息**，输入标题和端点URL。对于本教程，使用URL`http://localhost:5601/app/management/opensearch-dashboards/dataSources`。输入描述是可选的。

    - 在下面**身份验证方法**，从下拉列表中选择一个身份验证方法。选择身份验证方法后，将出现该方法的适用字段。然后，您可以输入所需的详细信息。身份验证方法选项是：
      - **没有身份验证**：没有使用身份验证来连接到数据源。
      - **用户名密码**：基本的用户名和密码用于连接到数据源。
      - **AWS SIGV4**：AWS签名版本4认证请求用于连接到数据源。AWS签名版本4需要一个访问密钥和秘密键。
            - 对于AWS签名版本4身份验证，首先指定**地区**。接下来，在**服务名称** 列表。选项是**Amazon OpenSearch服务** 和**Amazon OpenSearch无服务器**。最后，输入**访问密钥** 和**密钥** 用于授权。
      
      有关AWS帐户可用AWS区域的信息，请参见[可用地区](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions)。有关AWS签名版本4身份验证请求的更多信息，请参阅[认证请求（AWS签名版本4）](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html)。
      {: .note}

    - 在所有必需字段中输入适当的详细信息后，**测试连接** 和**创建数据源** 按钮变得活跃。您可以选择**测试连接** 确认连接有效。

3. 选择**创建数据源** 保存您的设置。连接是创建的。活动窗口返回到**数据源** 主页和新连接出现在数据源列表中。

4. 编辑或更新数据源连接。

    - 要更改数据源连接，请在列表中选择一个连接**数据源** 主页。这**连接详细信息** 窗口打开。

    - 进行更改**连接详细信息**，编辑一个或两个**标题** 和**描述** 字段和选择**保存更改** 在较低-屏幕的右角。您也可以在此处取消更改。更改**身份验证方法**，选择其他身份验证方法，输入您的凭据（如果适用），然后选择**保存更改** 在较低-屏幕的右角。更改已保存。

        - 什么时候**用户名密码** 是选定的身份验证方法，您可以通过选择来更新密码**更新存储的密码** 旁边**密码** 场地。在流行音乐中-向上窗口，在第一个字段中输入新密码，然后再次在第二个字段中输入以确认。选择**更新存储的密码** 在流行音乐中-向上窗口。新密码已保存。选择**测试连接** 确认连接有效。

        - 什么时候**AWS SIGV4** 是所选的身份验证方法，您可以通过选择来更新凭据**更新存储的AWS凭据**。在流行音乐中-向上窗口，在第一个字段中输入新的访问密钥，并在第二字段中输入一个新的秘密密钥。选择**更新存储的AWS凭据** 在流行音乐中-向上窗口。新的凭据被保存。选择**测试连接** 在上部-屏幕的右角确认连接有效。

5. 通过选择标题左侧的复选框删除数据源连接，然后选择**删除1连接**。支持为多个连接选择多个复选框。另外，选择垃圾桶图标（{:: nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/trash-can-icon.png" class ="inline-icon" alt ="trash can icon"/> {：/}）。

示例数据源连接屏幕如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/data-source-connection.png" alt="Data source connection screen" width="700">

### 通过开发工具控制台选择多个数据源

另外，您可以通过[开发工具]({{site.url}}{{site.baseurl}}/dashboards/dev-tools/index-dev/) 安慰。此选项提供了与更广泛的数据合作，并更深入地了解您的代码和应用程序。

观看以下10-第二个视频可以在行动中看到它。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/multidata-dev-tools.gif" alt="Multiple data sources in Dev Tools demo">{: .img-fluid}

要通过开发工具控制台选择数据源，请按照以下步骤：

1. 找到您的副本`opensearch_dashboards.yml` 并在您选择的编辑中打开它。
2. 放`data_source.enabled` 到`true`。
3. 连接到OpenSearch仪表板，然后选择**开发工具** 在菜单中。
4. 输入以下查询的编辑窗格**安慰** 然后选择“播放”按钮：

    ```json
    GET /_cat/indices
    ```
    {% include copy-curl.html %}}

5. 来自**数据源** 下拉菜单，选择一个数据源，然后查询源。
6. 重复您要选择的每个数据源的前面步骤。

## 下一步

配置了多个数据源后，您可以开始探索该数据。请参阅以下资源以了解更多信息：

- 学习关于[管理索引模式]({{site.url}}{{site.baseurl}}/dashboards/management/index-patterns/) 通过OpenSearch仪表板。
- 学习关于[使用索引管理索引数据]({{site.url}}{{site.baseurl}}/dashboards/im-dashboards/index/) 通过OpenSearch仪表板。
- 了解如何[通过OpenSearch仪表板连接OpenSearch和Amazon S3]({{site.url}}{{site.baseurl}}/dashboards/management/S3-data-source/)。
- 了解[集成工具]({{site.url}}{{site.baseurl}}/integrations/index/)，这使您可以灵活地使用各种数据摄入方法并连接仪表板UI的数据。

## 限制

此功能有一些局限性：

*索引支持多个数据源功能-图案-仅基于可视化。
*可视化类型的时间序列视觉构建器（TSVB），VEGA和VEGA-Lite和时间表不支持。
*外部插件，例如gantt图表和非-不支持可视化插件（例如开发人员控制台）。


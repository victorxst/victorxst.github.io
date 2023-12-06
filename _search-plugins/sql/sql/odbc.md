---
layout: default
title: ODBC驱动程序
parent: SQL
grand_parent: SQL和PPL
nav_order: 72
redirect_from:
  - /search-plugins/sql/odbc/
---

# ODBC驱动程序

开放数据库连接（ODBC）驱动程序是一个读取-仅用于Windows和MacOS的ODBC驱动程序，才能连接商业智能（BI）和数据可视化应用程序[Microsoft Excel](https://github.com/opensearch-project/sql-odbc/blob/main/docs/user/microsoft_excel_support.md) 和[Power BI](https://github.com/opensearch-project/sql-odbc/blob/main/bi-connectors/PowerBIConnector/README.md) 到SQL插件。

有关下载和使用驱动程序的信息，请参阅[GitHub上的SQL存储库](https://github.com/opensearch-project/sql-odbc)。

## 规格

ODBC驱动程序与ODBC版本3.51兼容。

## 支持的OS版本

支持以下操作系统：

操作系统| 版本
:--- | :---
视窗| Windows 10，Windows 11
苹果系统| Catalina 10.15.4，Mojave 10.14.6，Big Sur 11.6.7，蒙特雷12.4


## 概念

学期| 定义
:--- | :---
**DSN** | DSN（数据源名称）用于将驱动程序信息存储在系统中。通过将信息存储在系统中，无需每次驱动程序连接时指定信息。
**.tdc** 文件| TDC文件包含Tableau适用于与数据库供应商名称和驱动程序名称相匹配的任何连接的配置信息。这种配置使您可以罚款-调整ODBC数据连接的部分并打开/关闭数据源不支持的某些功能。


## 安装驱动程序

要安装驱动程序，请从[这里](https://opensearch.org/downloads.html) 或通过源头构建。


### 视窗

1. 打开下载`OpenSearch SQL ODBC Driver-<version>-Windows.msi` 安装程序。

   安装程序未签名，并显示一个安全对话框。选择**更多信息** 和**无论如何运行**。

2. 选择**下一个** 进行安装。

3. 接受协议，然后选择**下一个**。

4. 安装程序包含文档和有用的资源文件，以连接到各种BI工具（例如，一个`.tdc` 申请Tableau）。您可以选择保留或删除这些资源。选择**下一个**。

5. 选择**安装** 和**结束**。

作为默认DSN的一部分设置以下连接信息：

```
Host: localhost
Port: 9200
Auth: NONE
```

要自定义DSN，请使用**ODBC数据源管理员** 哪个是-安装在Windows 10上。


### 苹果系统

在MACOS上安装ODBC驱动程序之前，请安装IODBC驱动程序管理器。

1. 打开下载`OpenSearch SQL ODBC Driver-<version>-Darwin.pkg` 安装程序。

   安装程序未签名，并显示一个安全对话框。正确的-单击安装程序，然后选择**打开**。

2. 选择**继续** 几次进行安装。

3. 选择**目的地** 安装驱动程序文件。

4. 安装程序包含文档和有用的资源文件以连接到各种BI工具（例如，一个`.tdc` 申请Tableau）。您可以选择保留或删除这些资源。选择**继续**。

5. 选择**安装** 和**关闭**。

当前，DSN尚未作为安装的一部分设置，需要手动配置。首先，打开`iODBC Administrator`：

```
sudo /Applications/iODBC/iODBC\ Administrator64.app/Contents/MacOS/iODBC\ Administrator64
```

此命令允许应用程序权限保存驱动程序和DSN配置。

1. 选择**ODBC驱动程序** 标签。
2. 选择**添加驱动程序** 并填写以下详细信息：
   - **驾驶员的描述**：输入您用于ODBC连接的驱动程序名称（例如，OpenSearch SQL ODBC驱动程序）。
   - **驱动程序文件名**：输入驱动程序文件的路径（默认：`<driver-install-dir>/bin/libopensearchsqlodbc.dylib`）。
   - **设置文件名**：输入设置文件的路径（默认值：`<driver-install-dir>/bin/libopensearchsqlodbc.dylib`）。

3. 选择用户驱动程序。
4. 选择**好的** 保存选项。
5. 选择**用户DSN** 标签。
6. 选择**添加**。
7. 选择您上面添加的驱动程序。
8. 为了**数据源名称（DSN）**，输入用于存储连接选项的DSN的名称（例如，OpenSearch SQL ODBC DSN）。
9. 为了**评论**，添加可选评论。
10. 添加密钥-通过使用`+` 按钮。我们建议为默认的本地OpenSearch安装提供以下选项：
   - **主持人**：`localhost` - OpenSearch服务器端点
   - **港口**：`9200` - 服务器端口
   - **auth**：`NONE` - 身份验证模式
   - **用户名**：`(blank)` - 用于基本验证的用户名
   - **密码**：`(blank)`- 基本验证的密码
   - **响应时间out**：`10` - 等待服务器响应的秒数
   - **Usessl**：`0` - 请勿使用SSL进行连接

11. 选择**好的** 保存DSN配置。
12. 选择**好的** 退出IODBC管理员。


## 自定义ODBC驱动程序

驱动程序是库文件的形式：`opensearchsqlodbc.dll` 对于Windows和`libopensearchsqlodbc.dylib` 对于macos。

如果您使用ODBC兼容的BI工具，请参阅您的BI工具文档以配置新的ODBC驱动程序。
通常，所需的只是让BI工具知道驱动程序库文件的位置，然后使用它来设置数据库（即OpenSearch）连接。


### 连接字符串和其他设置

ODBC驱动程序使用ODBC连接字符串。
连接字符串是半隆-划界字符串指定可以用于连接的选项集。
通常，连接字符串将要么：
  - 指定包含PRE的数据源名称（DSN）-配置的选项集（`DSN=xxx;User=xxx;Password=xxx;`）。
  - 或者，使用字符串明确配置选项（`Host=xxx;Port=xxx;LogLevel=ES_DEBUG;...`）。

您可以使用DSN或连接字符串配置以下驱动程序选项：

所有选项名称都是案例-不敏感。
{: .note}


#### 基本选项

选项| 描述| 类型| 默认
:--- | :---
`DSN` | 您用于配置连接的数据源名称。| `string` | -
`Host / Server` | 目标群集的主机名或IP地址。| `string` | -
`Port` | OpenSearch集群的REST接口正在侦听的端口号。| `string` | -

#### 身份验证选项

选项| 描述| 类型| 默认
:--- | :---
`Auth` | 使用的身份验证机制。| `BASIC` （基本HTTP），`AWS_SIGV4` （AWS Auth），或`NONE` | `NONE`
`User / UID` | [`Auth=BASIC`]连接的用户名。| `string` | -
`Password / PWD` | [`Auth=BASIC`]连接的密码。| `string` | -
`Region` | [`Auth=AWS_SIGV4`]用于签署请求的区域。| `AWS region (for example, us-west-1)` | -

#### 高级选项

选项| 描述| 类型| 默认
:--- | :---
`UseSSL` | 是否在SSL/TLS上建立连接。| `boolean (0 or 1)` | `false (0)`
`HostnameVerification` | 指示是否应针对SSL/TLS连接执行证书主机名验证。| `boolean` （0或1）| `true (1)`
`ResponseTimeout` | 在几秒钟内等待主机的响应的最大时间。| `integer` | `10`

#### 记录选项

选项| 描述| 类型| 默认
:--- | :---
`LogLevel` | 驱动程序日志的严重性水平。| `LOG_OFF`，`LOG_FATAL`，`LOG_ERROR`，`LOG_INFO`，`LOG_DEBUG`，`LOG_TRACE`， 或者`LOG_ALL` | `LOG_WARNING`
`LogOutput` | 存储驱动程序日志的位置。| `string` | `WIN: C:\`，`MAC: /tmp`

您需要管理特权来更改记录选项。
{: .note}

## 连接到Tableau

pre-要求：

- 确保已经设置了DSN。
- 确保OpenSearch在_host_和_Port_上运行，如DSN中配置。
- 确保`.tdc` 被复制到`<user_home_directory>/Documents/My Tableau Repository/Datasources` 在MacOS和Windows中。

1. 启动Tableau。在下面**连接** 部分，转到**到服务器** 并选择**其他数据库（ODBC）**。

2. 在里面**DSN下降-向下**，选择您在上一组步骤中设置的OpenSearch DSN。您添加的选项将自动填写**连接属性**。

3. 选择**登入**。几秒钟后，Tableau连接到OpenSearch Server。连接后，您将被指向**数据源** 窗户。这**数据库** OpenSearch集群的名称已经填充了。
要列出所有索引，请单击下面的搜索图标**桌子**。

4. 通过将表拖到连接区域开始尝试数据。选择**现在更新** 或者**自动更新** 填充表数据。

在此处查看更多详细说明[GitHub存储库](https://github.com/opensearch-project/sql/blob/1.x/sql-odbc/docs/user/tableau_support.md)。

### 故障排除

**问题**

无法连接到服务器。

**解决方法**

这很可能是由于OpenSearch服务器未运行**主持人** 和**邮政** 在DSN中配置。
确认**主持人** 和**邮政** 正确，OpenSearch Server使用OpenSearch SQL插件运行。
也确保`.tdc` 使用安装程序下载的该文件正确地复制到`<user_home_directory>/Documents/My Tableau Repository/Datasources` 目录。

## 连接到Microsoft Power BI

跟着[安装说明](https://github.com/opensearch-project/sql-odbc/tree/main/bi-connectors/PowerBIConnector/README.md) 和[配置说明](https://github.com/opensearch-project/sql-odbc/blob/main/bi-connectors/PowerBIConnector/power_bi_support.md) 发表在GitHub存储库中。


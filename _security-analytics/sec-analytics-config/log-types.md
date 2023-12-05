---
layout: default
title: 支持的日志类型
parent: 设置安全分析
nav_order: 16
---


# 支持的日志类型

日志包含有关在整个系统及其单独部分中发生的事件的原始数据。从OpenSearch 2.11开始，将日志类型按类别进行分组，以帮助选择，过滤和搜索日志类型。

导航到**日志类型** 页面，选择**日志类型** 在下面**探测器** 在里面**安全分析** 导航菜单。该页面显示日志类型的名称，其描述，其类别，并标识是否是标准opensearch-定义的日志类型或自定义日志类型。下图显示了**日志类型** 登陆页面带有选定类别列的着陆页，**类别** 过滤器您可以通过类别来过滤列表。

<img src="{{site.url}}{{site.baseurl}}/images/Security/c-log-type.png" alt="The Log types landing page." width="85%">

下表显示了当前由安全分析摄入，映射和监视的日志类型。

| 类别| 日志类型| 描述| 
| ：--- |：--- |：--- |
| 访问管理| `Ad_ldap` | Active Directory日志跟踪LDAP查询，LDAP服务器的错误，超时事件和不安全的LDAP绑定。|
| 访问管理| `Apache_access` | Apache访问日志记录了Apache HTTP服务器处理的所有请求的数据。|
| 访问管理| `Okta` | OKTA记录了从一系列操作中记录OKTA事件的记录，例如下载导出文件，请求应用程序访问或撤销特权。|
| 申请| `GitHub` | GitHub日志监视由创建的工作流[github动作](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)。|
| 申请| `Gworkspace` | Google Workspace日志监视日志条目，例如管理员操作，组成员和组成员资格操作以及与登录有关的事件。|  
| 申请| `M365` | Microsoft 365审核日志收集了Microsoft 365的一系列数据，包括呼叫详细信息，性能数据，SQL Server，Security Everts和Access Control Activity的记录。|
| 云服务| `Azure` | Microsoft Azure日志可监视由Azure Cloud Services管理的云应用程序的日志数据。|
| 云服务| `CloudTrail` | AWS CloudTrail日志监视AWS CloudTrail帐户的事件。OpenSearch可以从两者中摄入CloudTrail日志数据[亚马逊简单存储服务](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) （Amazon S3）帐户和[亚马逊安全湖](https://docs.aws.amazon.com/security-lake/latest/userguide/what-is-security-lake.html) 服务帐户。| 
| 云服务| `S3` | Amazon S3日志跟踪请求访问S3存储桶的请求。|
| 网络活动| `Dns` | DNS日志存储DNS活动。|
| 网络活动| `Network` | 网络日志记录在系统网络中发生的事件，例如登录尝试和应用程序事件。| 
| 网络活动| `vpcflow` | [VPC流日志](https://docs.aws.amazon.com/prescriptive-guidance/latest/logging-monitoring-for-application-owners/vpc-flow-logs.html) 该捕获有关IP流量访问和从虚拟私有云（VPC）中的网络接口的信息。|
| 安全| `Waf` | Web应用程序防火墙（WAF）日志（在OpenSearch 2.11中引入），用于需要监视具有安全分析的WAF用例的用户。WAF的作用是监视和过滤Web应用程序和Internet之间的HTTP流量。WAF阻止了常见的安全攻击，例如交叉-站点脚本（XSS）和SQL注入（SQI）。|
| 系统活动| `Linux` | Linux系统日志记录Linux Syslog事件。|
| 系统活动| `Windows` | Windows日志记录在操作系统，应用程序和其他Windows系统服务中发生的事件。|
| 其他| `Email` | 记录记录电子邮件活动的记录。|


## 页面动作

以下列表描述了在**日志类型** 页面和您可以采取的操作：

*选择日志类型**姓名** 打开日志类型的详细信息页面。这**细节** 默认情况下显示选项卡。此选项卡包括日志类型的ID。您也可以选择**检测规则** 选项卡以显示与日志类型关联的所有检测规则。
* 在里面**动作** 列，您可以选择垃圾桶图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/alerting/trash-can-icon.png" class ="inline-icon" alt ="trash can icon"/> {:/}）要删除自定义日志类型（您无法删除标准openSearch-定义的日志类型）。请按照提示确认并安全删除自定义日志类型。
* 选择**创建日志类型** 在上部-屏幕的右角开始创建自定义日志类型。这**创建日志类型** 页面打开。继续执行以下部分创建自定义日志类型的步骤。
* 使用**类别** 和**来源** 下拉阶段，您可以分别按日志类型类别或源进行排序。

## 相关文章
[创建自定义日志类型]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/custom-log-type/)




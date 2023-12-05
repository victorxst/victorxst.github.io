---
layout: default
title: 与Cron实用程序计划报告
nav_order: 20
parent: 使用CLI报告
grand_parent: 报告
redirect_from:
  - /dashboards/reporting-cli/rep-cli-cron/
---

# 与Cron实用程序计划报告

您可以使用cron命令-线路实用程序启动报告请求，并在任何日期或时间间隔定期运行的报告CLI。请按照cron表达式语法指定要启动的命令之前的日期和时间。

要了解Cron表达语法，请参阅[cron表达参考]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/cron/)。要获得有关Cron的帮助，请通过运行以下命令打开MAN页面：

```
man cron
```

### 先决条件

- 您需要安装Cron的机器。
- 您需要安装报告CLI。看[下载和安装报告CLI工具]({{site.url}}{{site.baseurl}}/dashboards/reporting-cli/rep-cli-install/)

## 指定报告详细信息

通过运行以下命令打开crontab编辑器：

```
crontab -e
```
在Crontab编辑器中，输入报告请求。以下示例显示了每天上午8:00的Cron报告：

```
0 8 * * * opensearch-reporting-cli -u https://playground.opensearch.org/app/dashboards#/view/084aed50-6f48-11ed-a3d5-1ddbf0afc873 -e ses -s <sender_email> -r <recipient_email>
```

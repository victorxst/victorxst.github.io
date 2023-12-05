---
layout: default
title: 下载并安装报告CLI工具
nav_order: 10
parent: 使用CLI报告
grand_parent: 报告
redirect_from:
  - /dashboards/reporting-cli/rep-cli-install/
---

# 下载并安装报告CLI工具

您可以从NPM软件注册表或OpenSearch.org下载并安装报告CLI工具[文物](https://opensearch.org/artifacts) 中心。有关说明，请参阅以下各节。

要了解有关NPM软件注册表的更多信息，请参阅[NPM](https://docs.npmjs.com/about-npm) 文档。

## 从NPM下载和安装报告CLI

要从NPM下载并安装报告CLI，请运行以下命令以启动安装：

```
npm i @opensearch-project/reporting-cli
```

## 从opensearch.org下载和安装报告CLI

您可以下载`opensearch-reporting-cli` opensearch.org的工具[文物](https://artifacts.opensearch.org/reporting-cli/opensearch-reporting-cli-1.0.0.tgz) 中心。

接下来，运行以下命令安装.tar存档：

```
npm install -g opensearch-reporting-cli-1.0.0.tgz
```

为了为工件提供更好的安全性，我们建议您通过下载来验证签名[报告CLI签名文件](https://artifacts.opensearch.org/reporting-cli/opensearch-reporting-cli-1.0.0.tgz.sig)。
{: .important }

要了解有关验证签名的更多信息，请参阅[如何验证可下载工件的签名](https://opensearch.org/verify-signatures.html)


---
layout: default
title: OpenSearch CLI
nav_order: 70
has_children: false
redirect_from:
  - /clients/cli/
---

# OpenSearch CLI

OpenSearch CLI命令行接口（OpenSearch-CLI）可让您从命令行管理OpenSearch Cluster并自动化任务。

目前，OpenSearch-CLI支持[异常检测]({{site.url}}{{site.baseurl}}/monitoring-plugins/ad/) 和[k-nn]({{site.url}}{{site.baseurl}}/search-plugins/knn/) 插件，以及任意的REST API路径。除其他外，您可以使用OpenSearch-CLI创建和删除检测器，启动和停止它们，然后检查K-NN统计。

配置文件可让您轻松访问不同的簇或具有不同凭据的请求。OpenSearch-CLI支持未经身份验证的请求，HTTP基本签名以及Amazon Web服务的IAM签名。

此示例移动检测器（`ecommerce-count-quantity`）从分期集群到生产集群：

```bash
opensearch-cli ad get ecommerce-count-quantity --profile staging > ecommerce-count-quantity.json
opensearch-cli ad create ecommerce-count-quantity.json --profile production
opensearch-cli ad start ecommerce-count-quantity.json --profile production
opensearch-cli ad stop ecommerce-count-quantity --profile staging
opensearch-cli ad delete ecommerce-count-quantity --profile staging
```


## 安装

1. [下载](https://opensearch.org/downloads.html){：target ='\ _ blank'}并为计算机提取适当的安装程序包。

1. 做`opensearch-cli` 文件可执行文件：

   ```bash
   chmod +x ./opensearch-cli
   ```

1. 将命令添加到您的路径：

   ```bash
   export PATH=$PATH:$(pwd)
   ```

1. 确认CLI正常工作：

   ```bash
   opensearch-cli --version
   ```


## 概况

配置文件可让您轻松地在不同的群集和用户凭据之间切换。首先，运行`opensearch-cli profile create` 与`--auth-type`，`--endpoint`， 和`--name` 选项：

```bash
opensearch-cli profile create --auth-type basic --endpoint https://localhost:9200 --name docker-local
```

或者，将配置文件保存到`~/.opensearch-cli/config.yaml`：

```yaml
profiles:
    - name: docker-local
      endpoint: https://localhost:9200
      user: admin
      password: foobar
    - name: aws
      endpoint: https://some-cluster.us-east-1.es.amazonaws.com
      aws_iam:
        profile: ""
        service: es
```


## 用法

OpenSearch-CLI命令使用以下语法：

```bash
opensearch-cli <command> <subcommand> <flags>
```

例如，以下命令检索有关检测器的信息：

```bash
opensearch-cli ad get my-detector --profile docker-local
```

对于OpenSearch CAT API的请求，请尝试以下命令：

```bash
opensearch-cli curl get --path _cat/plugins --profile aws
```

使用`-h` 或者`--help` 标志以查看所有受支持的命令，子命令或特定命令的用法：

```bash
opensearch-cli -h
opensearch-cli ad -h
opensearch-cli ad get -h
```


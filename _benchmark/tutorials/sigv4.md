---
layout: default
title: AWS签名版本4支持
nav_order: 70
parent: 教程
---

# 使用AWS签名版本4运行OpenSearch Benchmark

OpenSearch基准测试支持AWS签名版本4身份验证。要使用签名版本4运行基准测试，请使用以下步骤：

1. 设置一个[IAM用户或IAM角色](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html) 并使用签名版本4身份验证提供对OpenSearch集群的访问。

2. 为您的IAM用户设置以下环境变量：

   ```bash
   导出OSB_AWS_ACCESS_KEY_ID = <IAM用户AWS访问密钥ID>
   导出OSB_AWS_SECRET_ACCESS_KEY = <IAM用户AWS秘密访问键>
   导出osb_region = <您的区域>
   导出OSB_Service = ES
   ```
   {% include copy.html %}

   If you want to set up an IAM role instead of an IAM user, use the following environment variables instead:

   ```bash
   导出OSB_AWS_ACCESS_KEY_ID = <IAM角色AWS访问密钥ID>
   导出OSB_AWS_SECRET_ACCESS_KEY = <IAM角色AWS秘密访问键>
   导出osb_aws_session_token = <IAM角色ersect token>
   导出osb_region = <您的区域>
   导出OSB_Service = ES
   ```
   {% include copy.html %}

  If you're testing against Amazon OpenSearch Serverless, set `OSB_Service` to `aoss`.

3. Customize and run the following `执行-测试` command with the ` --客户-选项= Amazon_AWS_LOG_IN：环境` flag. This flag tells OpenSearch Benchmark the location of your exported credentials.

   ```bash
   OpenSearch-基准执行-测试 \
   --目标-hosts = <cluster endpoint> \
   --管道=基准-仅有的 \
   --Workload = GeOnames \
   --客户-选项=超时：120，Amazon_AWS_LOG_IN：环境\
   ```


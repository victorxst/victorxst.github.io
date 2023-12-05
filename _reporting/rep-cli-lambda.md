---
layout: default
title: 与AWS Lambda计划报告
nav_order: 30
parent: 使用CLI报告
grand_parent: 报告
redirect_from:
  - /dashboards/reporting-cli/rep-cli-lambda/
---

# 与AWS lambda进行计划报告

您可以将AWS Lambda与报告CLI工具一起指定AWS lambda功能以触发报告生成。

这要求您使用AMD64系统和Docker。

### 先决条件

要将报告CLI与AWS Lambda一起使用，您需要执行以下初步步骤。

- 获取一个AWS帐户。有关说明，请参阅[创建一个AWS帐户](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html) 在AWS帐户管理参考指南中。
- 设置Amazon弹性容器注册表（ECR）。有关说明，请参阅[使用AWS管理控制台开始使用Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-console.html)。

## 步骤1：用Dockerfile创建一个容器图像

您需要通过运行DockerFile来组装容器图像。运行Dockerfile时，它下载了使用报告CLI所需的OpenSearch文物。要了解有关Dockerfiles的更多信息，请参阅[Dockerfile参考](https://docs.docker.com/engine/reference/builder/)。

将以下示例配置复制到dockerfile：

```dockerfile
# Define function directory
ARG FUNCTION_DIR="/function"

# Base image of the docker container
FROM node:lts-slim as build-image

# Include global arg in this stage of the build
ARG FUNCTION_DIR

# AWS Lambda runtime dependencies
RUN apt-get update && \
    apt-get install -y \
        g++ \
        make \
        unzip \
        libcurl4-openssl-dev \
        autoconf \
        automake \
        libtool \
        cmake \
        python3 \
        libkrb5-dev \
        curl

# Copy function code
WORKDIR ${FUNCTION_DIR}
RUN npm install @opensearch-project/reporting-cli && npm install aws-lambda-ric

# Build Stage 2: Copy Build Stage 1 files in to Stage 2. Install chrome, then remove chrome to keep the dependencies.
FROM node:lts-slim
# Include global arg in this stage of the build
ARG FUNCTION_DIR
# Set working directory to function root directory
WORKDIR ${FUNCTION_DIR}
# Copy in the build image dependencies
COPY --from=build-image ${FUNCTION_DIR} ${FUNCTION_DIR}

# Install latest chrome dev package and fonts to support major char sets (Chinese, Japanese, Arabic, Hebrew, Thai and a few others)
# Note: this installs the necessary libs to make the bundled version of Chromium that Puppeteer installs, work.
RUN apt-get update \
    && apt-get install -y wget gnupg \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 \
      --no-install-recommends \
    && apt-get remove -y google-chrome-stable \
    && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["/usr/local/bin/npx", "aws-lambda-ric"]

ENV HOME="/tmp"
CMD [ "/function/node_modules/@opensearch-project/reporting-cli/src/index.handler" ]

```

接下来，在包含DockerFile的同一目录中运行以下构建命令：

```
docker build -t opensearch-reporting-cli .
```

## 步骤2：使用Amazon ECR创建一个私人存储库

您需要按照说明来创建图像存储库，请参阅[使用AWS管理控制台开始使用Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-console.html)。

给您的存储库名称`opensearch-reporting-cli`。

除了Amazon ECR指令外，您还需要对报告CLI进行多次调整，以正常运行，如以下步骤中所述。

## 步骤3：将图像推到私人存储库

您需要从AWS ECR控制台中获取多个命令才能在DockerFile目录中运行。

1. 创建存储库后，从**私人存储库**。
1. 选择**查看推送命令**。
1. 复制并运行显示的每个命令**推送命令进行openSearch-报告-CLI** 依次在Dockerfile目录中。

有关Docker推送命令的更多详细信息，请参阅[推出码头图像](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html) 在《亚马逊ECR用户指南》中。

## 步骤4：使用容器图像创建lambda功能

现在，您已经为报告CLI创建了一个容器映像，需要创建一个定义为容器映像的函数。

1. 打开AWS Lambda控制台并选择[功能](https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/functions)。
1. 选择**创建功能**，然后选择**容器图像** 并填写功能的名称。
1. 在**容器图像URI**， 选择**浏览图像** 并选择`opensearch-reporting-cli` 对于图像存储库。
1. 在**图片** 选择图像，然后选择**选择图像**。
1. 在**建筑学**， 选择**x86_64**。
1. 选择**创建功能**。
1. 去**Lambda** >**功能** 并选择您创建的功能。
1. 选择**配置>常规配置>编辑超时** 并将Lambda的超时设置为5分钟，以允许报告CLI生成报告。
1. 更改**短暂存储** 设置为至少1024MB。默认设置不足以支持报告生成。

1. 接下来，通过提供值JSON格式或提供AWS lambda环境变量来测试函数。

- 如果该功能包含固定值，例如电子邮件地址，则不需要JSON文件。您可以在AWS lambda中指定环境变量。
- 如果功能采用变量键-值对，然后您需要用与命令选项相同的命名约定指定JSON中的值，例如`--credentials` 选项需要用户名和密码。
{： 。笔记 }

 以下示例显示了为发送者和收件人电子邮件地址提供的固定值：

```json
{
  "url": "https://playground.opensearch.org/app/dashboards#/view/084aed50-6f48-11ed-a3d5-1ddbf0afc873",
  "transport": "ses",
  "from": "sender@amazon.com", 
  "to": "recipient@amazon.com", 
  "subject": "Test lambda docker image"
}
```

要了解有关AWS Lambda功能的更多信息，请参阅[部署Lambda充当容器图像](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-images.html) 在AWS Lambda文档中。
## 步骤5：添加触发器以启动AWS lambda功能

设置扳机以开始运行报告。AWS Lambda可以将任何AWS服务用作触发器，例如SNS，S3或AWS CloudWatch Eventbridge。

1. 在里面**触发器** 部分，选择**添加触发器**。
1. 从列表中选择一个触发器。例如，您可以设置AWS CloudWatch事件。要了解有关亚马逊ECR活动的更多信息，请参阅[亚马逊ECR的样本事件](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr-eventbridge.html#ecr-eventbridge-bus)。
1. 选择**测试** 启动功能。

## （可选）步骤6：添加亚马逊SES的角色权限

如果您想将Amazon SES用于电子邮件运输，则需要设置权限。

1. 选择**配置** 并选择**执行角色**。
1. 在**概括**， 选择**权限**。
1. 选择**{} JSON** 打开JSON政策编辑。
1. 添加要使用的Amazon SES资源的权限。

以下示例为发送电子邮件措施提供了资源：

```json
{
"Effect": "Allow",
"Action": [
      "ses:SendEmail",
      "ses:SendRawEmail"
            ],
"Resource": "arn:aws:ses:us-west-2:555555511111:identity/username@amazon.com"
}
```

要了解有关设定角色权限的更多信息，请参阅[权限](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-images.html#gettingstarted-images-permissions) 在AWS Lambda用户指南中


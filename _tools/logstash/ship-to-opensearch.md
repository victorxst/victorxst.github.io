---
layout: default
title: 将事件运送到OpenSearch
parent: Logstash
nav_order: 220
redirect_from:
 - /clients/logstash/ship-to-opensearch/
---

# 将事件运送到OpenSearch

您可以将LogStash事件运送到OpenSearch Cluster，然后使用OpenSearch仪表板可视化事件。

确保你有[logstash]({{site.url}}{{site.baseurl}}/tools/logstash/index#install-logstash)，[OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/)， 和[OpenSearch仪表板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)。
{: .note }

## OpenSearch输出插件

要运行OpenSearch输出插件，请在您的`pipeline.conf` 文件：

```yml
output {
  opensearch {
    hosts       => "https://localhost:9200"
    user        => "admin"
    password    => "admin"
    index       => "logstash-logs-%{+YYYY.MM.dd}"
    ssl_certificate_verification => false
  }
}
```


## 样品演练

1.  打开`config/pipeline.conf` 文件并添加以下配置：

    ```yml
    input {
      stdin {
        codec => json
      }
    }

    output {
      opensearch {
        hosts       => "https://localhost:9200"
        user        => "admin"
        password    => "admin"
        index       => "logstash-logs-%{+YYYY.MM.dd}"
        ssl_certificate_verification => false
      }
    }
    ```

    该LogStash管道通过终端接受JSON输入，并将事件运送到本地运行的OpenSearch集群。logstash将事件写入索引`logstash-logs-%{+YYYY.MM.dd}` 命名约定。

2. 启动logstash：

    ```bash
    $ bin/logstash -f config/pipeline.conf --config.reload.automatic
    ```

    `config/pipeline.conf` 是通往`pipeline.conf` 文件。您也可以使用绝对路径。

3. 在终端中添加JSON对象：

    ```json
    { "amount": 10, "quantity": 2}
    ```

4. 启动OpenSearch仪表板并选择**开发工具**：

    ```json
    GET _cat/indices?v

    health | status | index | uuid | pri | rep | docs.count | docs.deleted | store.size | pri.store.size
    green | open | logstash-logs-2021.07.01 | iuh648LYSnmQrkGf70pplA | 1 | 1 | 1 | 0 | 10.3kb | 5.1kb
    ```

## 在输出插件中添加不同的身份验证机制

## auth_type支持不同的身份验证机制

除了现有的身份验证机制外，如果要添加新的身份验证，我们将通过使用auth_type将它们添加到配置中

基本身份验证的示例配置：

```yml
output {    
    opensearch {        
          hosts  => ["https://hostname:port"]     
          auth_type => {            
              type => 'basic'           
              user => 'admin'           
              password => 'admin'           
          }             
          index => "logstash-logs-%{+YYYY.MM.dd}"       
   }            
}               
```
### auth_type中的参数

- 类型（字符串）- 我们应该指定身份验证的类型
- 我们应该添加该身份验证所需的凭据，例如“用户”和“基本”身份验证
- 我们还应该添加该身份验证机制所需的其他参数

## AWS IAM身份验证的配置

要使用AWS_IAM身份验证运行LogStash输出opensearch插件，只需在以下文档之后添加配置即可。

示例配置：

```yml
output {        
   opensearch {     
          hosts => ["https://hostname:port"]              
          auth_type => {    
              type => 'aws_iam'     
              aws_access_key_id => 'ACCESS_KEY'     
              aws_secret_access_key => 'SECRET_KEY'     
              region => 'us-west-2'    
              service_name => 'es'     
          }         
          index  => "logstash-logs-%{+YYYY.MM.dd}"      
   }            
}
```

### 必需的参数

- 主机（字符串数组）- AmazonOpenSearchService域端点：端口号
- auth_type（json对象）- 它拥有身份验证所需的其他参数
    - 类型（字符串）- "aws_iam"
    - AWS_ACCESS_KEY_ID（字符串）- AWS访问密钥
    - aws_secret_access_key（字符串）- AWS秘密访问密钥
    - 区域（字符串，：Default =>"us-east-1"）- 域所在的区域
    - 如果我们要传递其他可选参数，例如配置文件，session_token等。它们需要在auth_type中添加
- 端口（字符串）- HTTPS的AmazonOpenSearchService在端口443上听
- 协议（字符串）- 用于连接到AmazonOpenSearchService的协议是“ HTTPS”

### 可选参数
- 凭证分辨率逻辑可以描述如下：
    - 用户通过aws_access_key_id and aws_secret_acre_key在配置中
    - 环境变量- AWS_ACCESS_KEY_ID和AWS_SECRET_ACCESS_KEY（推荐，因为它们均被所有AWS SDK和CLI识别），或AWS_ACCESS_KEY和AWS_SECRET_KEY（仅Java SDK识别）
    - 所有AWS SDK和AWS CLI共享的默认位置（〜/.aws/recertentials）处的凭证配置文件文件
    - 通过亚马逊EC2元数据提供的实例配置文件凭据
- 模板（路径）- 您可以在此处设置通往自己模板的路径。如果未指定模板，则插件使用默认模板。
- template_name（字符串，默认值=>"logstash"）- 定义模板在OpenSearch中的命名
- service_name（字符串，默认值=>"es"）- 定义要使用的服务名称`aws_iam` 验证。
- LEGACY_TEMPLATE（布尔值，默认值=> true）- 选择OpenSearch模板API。什么时候`true`，通过_template API使用旧模板。什么时候`false`，通过_INDEX_TEMPLATE API使用可复合模板。
- default_server_major_version（编号）- OpenSearch Server专业版本将在OpenSearch root URL中获得时使用。如果未设置，则当无法获取版本时，插件会引发异常。

## 数据流

OpenSearch输出插件可以存储时间序列数据集（例如日志，事件和指标）和非-OpenSearch中的时间序列数据。
建议将数据流索引时间序列数据集（例如日志，指标和事件）中搜索。

要了解有关数据流的更多信息，请参阅此信息[文档](https://opensearch.org/docs/latest/opensearch/data-streams/)。

我们可以通过LogStash将数据摄入数据流。我们需要创建数据流并指定数据流的名称和`op_type` 的`create` 在输出配置中。样本配置如下：

```yml
output {    
    opensearch {        
          hosts  => ["https://hostname:port"]     
          auth_type => {            
              type => 'basic'           
              user => 'admin'           
              password => 'admin'           
          }
          index => "my-data-stream"
          action => "create"
   }            
}               
```


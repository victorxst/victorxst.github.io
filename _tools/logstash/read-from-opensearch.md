---
layout: default
title: 从OpenSearch中阅读
parent: Logstash
nav_order: 220
redirect_from:
 - /clients/logstash/ship-to-opensearch/
---

# 从OpenSearch中阅读

当我们将LogStash事件运送到OpenSearch集群时[OpenSearch输出插件](https://github.com/opensearch-project/logstash-output-opensearch)，我们还可以在OpenSearch集群上执行读取操作，并使用该数据将数据加载到Logstash中[OpenSearch输入插件](https://github.com/opensearch-project/logstash-input-opensearch)。

OpenSearch Input插件读取在OpenSearch集群上执行的搜索查询结果，并将其加载到LogStash中。这使您可以根据加载数据重播测试日志，重新指示并执行其他操作。您可以通过使用
[克朗表达](https://opensearch.org/docs/latest/monitoring-plugins/alerting/cron/)，或通过运行查询一次手动将数据加载到logstash中。



## OpenSearch输入插件

要运行OpenSearch输入插件，请将配置添加到`pipeline.conf` 在您的logstash中文件`config` 文件夹。下面的示例运行`match_all` 查询过滤器并在数据中加载一次。

```yml
input {
  opensearch {
    hosts       => "https://hostname:port"
    user        => "admin"
    password    => "admin"
    index       => "logstash-logs-%{+YYYY.MM.dd}"
    query       => '{ "query": { "match_all": {}} }'
  }
}

filter {
}

output {
}
```

要根据时间表摄入数据，请使用指定您想要的时间表的CRON表达式。例如，要每分钟加载数据，请添加`schedule => "* * * * *"` 到您的输入部分`pipeline.conf` 文件。

像输出插件一样，将您的配置添加到`pipeline.conf` 文件，通过提供该文件的路径开始logStash：

 ```bash
 $ bin/logstash -f config/pipeline.conf --config.reload.automatic
 ```

`config/pipeline.conf` 是通往`pipeline.conf` 文件。您也可以使用绝对路径。

添加`stdout{}` 到`output{}` 您的部分`pipeline.conf` 文件将查询结果打印到控制台。

要将数据索引到OpenSearch域，请在中添加目标域配置`output{}` 如图所示[这里](https://opensearch.org/docs/latest/tools/logstash/index/)。


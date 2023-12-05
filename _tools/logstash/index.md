---
layout: default
title: Logstash
nav_order: 150
has_children: true
has_toc: true
redirect_from:
  - /clients/logstash/
  - /clients/logstash/index/
---

# logstash

logstash是真实的-时间事件处理引擎。它是OpenSearch堆栈的一部分，其中包括OpenSearch，Beats和OpenSearch仪表板。

您可以将事件从许多不同来源发送到LogStash。LogStash处理事件并将其发送一个或多个目的地。例如，您可以将访问日志从Web服务器发送到LogStash。LogStash从每个日志中提取有用的信息，并将其发送到OpenSearch之类的目的地。

将事件发送到LogStash，可以使您可以从应用程序中切换事件处理。您的应用只需要将事件发送到LogStash，而无需了解事后发生的事情。

开放-源社区最初构建了用于处理日志数据的LogStash，但现在您可以处理任何类型的事件，包括XML或JSON格式的事件。

## 管道结构

LogStash的工作方式是您配置具有三个阶段⁠的管道---输入，过滤器和输出。

每个阶段都使用一个或多个插件。Logstash已建造了200多个-在插件中，您可能会找到所需的东西。除了建造-在插件中，您可以使用社区的插件，甚至可以写自己的插件。

管道的结构如下：

```yml
input {
  input_plugin => {}
}

filter {
  filter_plugin => {}
}

output {
  output_plugin => {}
}
```

在哪里：

*`input` 同时接收来自多个来源的日志之类的事件。LogStash支持用于TCP/UDP，文件，Syslog，Microsoft Windows EventLogs，STDIN，HTTP等的许多输入插件。您还可以使用称为BEATS的输入工具的开源收集来收集事件。输入插件将事件发送到过滤器。
*`filter` 以一种或另一种方式解析和丰富事件。LogStash有大量的过滤器插件，可修改事件并将其传递给输出。例如，`grok` 过滤器将非结构化事件解析到字段和一个`mutate` 过滤器更改字段。过滤器依次执行。
*`output` 将过滤事件运送到一个或多个目的地。LogStash支持诸如OpenSearch，TCP/UDP，电子邮件，文件，Stdout，HTTP，Nagios等目的地的各种输出插件。

输入和输出阶段都支持编解码器进入或退出管道时处理事件。
一些流行的编解码器是`json` 和`multiline`。这`json` 编解码器处理JSON格式的数据和`multiline` 编解码器将多个线路事件合并为一行。

您还可以在管道配置中编写条件语句，以执行某些操作，如果满足某些条件。

## 安装logstash

要在OpenSearch上安装LogStash，请先在群集上安装LogStash，然后在以下步骤中所述的OpenSearch LogStash插件。

### tarball

确保你有[Java开发套件（JDK）](https://www.oracle.com/java/technologies/javase-downloads.html) 安装了8或11版。

1. 从中下载logstash tarball[LogStash下载](https://www.elastic.co/downloads/logstash)。

2. 导航到终端中下载的文件夹并提取文件。确保您的logstash版本和平台与下载的版本匹配：

     ```bash
     tar -zxvf logstash-8.8.2-linux-x86_64.tar.gz
     ```

3. 导航到`logstash-8.8.2` 目录。

4. 使用以下命令安装插件：

     ```bash
     bin/logstash-插件安装logstash-输出-OpenSearch
     ```
  
   You should receive the following output:

     ```
     验证logstash-输出-OpenSearch
     解决混合蛋白依赖性
     更新Mixin依赖项logstash-混合蛋白-ecs_compatibility_support
     Bundler试图更新Logstash-混合蛋白-ecs_compatibility_support，但其版本保持不变
     安装logstash-输出-OpenSearch
     安装成功
     ```

You can add your pipeline configurations to the `config` directory. Logstash saves any data from the plugins in the `数据` directory. The `垃圾桶` 目录包含用于启动LogStash和托管插件的二进制文件。

### Docker

1. 按照在[LogStash下载](https://www.elastic.co/downloads/logstash)。

    ```
    docker pull docker.elastic.co/logstash/logstash:8.8.2
    ```

1. 创建一个Docker网络：

    ```
    docker network create test
    ```

1. 使用此网络开始开放搜索：

    ```
    docker run -p 9200:9200 -p 9600:9600 --name opensearch --net test -e "discovery.type=single-node" opensearchproject/opensearch:1.2.0
    ```

1. 启动logstash：

    ```
    Docker Run-它--R M--名称logstash--网络测试OpenSearchProject/logstash-OSS-和-OpenSearch-输出-插件：7.16.2-e'输入{stdin {}} output {
      OpenSearch {
        主机=> ["https://opensearch:9200"这是给出的
        索引=>"opensearch-logstash-docker-%{+YYYY.MM.dd}"
        用户=>"admin"
        密码=>"admin"
        SSL => true
        ssl_certificate_verification => false
      }
    }'
    ```

## Process text from the terminal

You can define a pipeline that listens for events on `斯丁` and outputs events on `Stdout`. `斯丁` and `Stdout` 请参阅您正在运行的LogStash的终端。

要在终端中输入一些文本，并在输出中查看事件数据：

1. 使用`-e` 参数将管道配置直接传递到Logstash二进制文件。在这种情况下，`stdin` 是输入插件和`stdout` 是输出插件：

    ```bash
    bin/logstash -e "input { stdin { } } output { stdout { } }"
    ```
    添加`—debug` 标志以查看更详细的输出。

2. 进入"hello world" 在您的航站楼。LogStash处理文本并将其输入终端：

    ```YML
    {
     "message" =>"hello world"，，，，
     "host" =>"a483e711a548.ant.amazon.com"，，，，
     "@timestamp" => 2021-05-30T05：15：56.816Z，
     "@version" =>"1"
    }
    ```

    The `信息` field contains your raw input. The `主持人` field is an IP address when you don’t run Logstash locally. `@timestamp` shows the date and time for when the event is processed. Logstash uses the `@版本` field for internal processing.

3. Press `Ctrl + c` to shut down Logstash.

### Troubleshooting

If you already have a Logstash process running, you’ll get an error. To fix this issue:

1. Delete the `。锁` file from the `数据` 目录：

    ```bash
    cd data
    rm -rf .lock
    ```

2. 重新启动logstash。

## 处理JSON或HTTP输入并将其输出到文件

定义处理JSON请求的管道：

1. 打开`config/pipeline.conf` 在您喜欢的任何文本编辑器中归档。您可以使用任何扩展名创建一个管道配置文件`.conf` 扩展是logStash约定。添加`json` 编解码器接受JSON作为输入和`file` 插件将处理的事件输出到`.txt` 文件：

    ```YML
    输入 {
      stdin {
        编解码器=> JSON
      }
    }
    输出 {
      文件 {
        路径=>"output.txt"
      }
    }
    ```

    To process inputs from a file, add an input file to the `事件-数据` directory and then pass its path to the `文件` plugin at the input:

    ```YML
    输入 {
      文件 {
        路径=>"events-data/input_data.log"
      }
    }
    ```

2. Start Logstash:

    ```bash
    $ bin/logstash-f config/pipeline.conf
    ```

    `config/pipeline.conf` is a relative path to the `pipeline.conf` 文件。您也可以使用绝对路径。

3. 在终端中添加JSON对象：

    ```json
    { "amount": 10, "quantity": 2}
    ```

    管道仅处理一行输入。如果粘贴一些跨越多行的JSON，则会出现错误。

4. 检查JSON对象的字段是否添加到`output.txt` 文件：

    ```JSON
    $ cat Uptuct.txt

    {
      "@version"："1"，，，，
      "@timestamp"："2021-05-30T05:52:52.421Z"，，，，
      "host"："a483e711a548.ant.amazon.com"，，，，
      "amount"：10，
      "quantity"：2
    }
    ```

    If you type in some invalid JSON as the input, you'll see a JSON parsing error. Logstash doesn't discard the invalid JSON because you still might want to do something with it. For example, you can trigger an email or send a notification to a Slack channel.

To define a pipeline that handles HTTP requests:

1. Use the `http` plugin to send events to Logstash through HTTP:

    ```YML
    输入 {
      http {
        主机=>"127.0.0.1"
        端口=> 8080
      }
    }

    输出 {
      文件 {
        路径=>"output.txt"
      }
    }
    ```

    If you don’t specify any options, the `http` plugin binds to `Localhost` and listens on port 8080.

2. Start Logstash:

    ```bash
    $ bin/logstash-f config/pipeline.conf
    ```

3. Use Postman to send an HTTP request. Set `内容-类型` to an HTTP header with a value of `应用程序/JSON`:

    ```JSON
    放127.0.0.1：8080

    {
      "amount"：10，
      "quantity"：2
    }
    ```

    Or, you can use the `卷曲` command:

    ```bash
    卷曲-xput-H"Content-Type: application/json" -d'{"amount"：7，"quantity"：3}'http：// localhost：8080（http：// localhost：8080/）
    ```

    Even though we haven't added the `JSON` plugin to the input, the pipeline configuration still works because the HTTP plugin automatically applies the appropriate codec based on the `内容-类型` header.
    If you specify a value of `应用程序/JSON`, Logstash parses the request body as JSON.

    The `标题` field contains the HTTP headers that Logstash receives:

    ```JSON
    {
      "host"："127.0.0.1"，，，，
      "quantity"："3"，，，，
      "amount"：10，
      "@timestamp"："2021-05-30T06:05:48.135Z"，，，，
      "headers"：{
        "http_version"："HTTP/1.1"，，，，
        "request_method"："PUT"，，，，
        "http_user_agent"："PostmanRuntime/7.26.8"，，，，
        "connection"："keep-alive"，，，，
        "postman_token"："c6cd29cf-1b37-4420-8db3-9faec66b9e7e"，，，，
        "http_host"："127.0.0.1:8080"，，，，
        "cache_control"："no-cache"，，，，
        "request_path"："/"，，，，
        "content_type"："application/json"，，，，
        "http_accept"："*/*"，，，，
        "content_length"："41"，，，，
        "accept_encoding"："gzip, deflate, br"
      }，，
    "@version"："1"
    }
    ```


## Automatically reload the pipeline configuration

You can configure Logstash to detect any changes to the pipeline configuration file or the input log file and automatically reload the configuration.

The `斯丁` plugin doesn’t supporting automatic reloading.
{: .note }

1. Add an option named `start_position` with a value of `开始` to the input plugin:

    ```YML
    输入 {
      文件 {
        路径=>"/Users/<user>/Desktop/logstash7-12.1/events-data/input_file.log"
        start_position =>"beginning"
      }
    }
    ```

    Logstash only processes any new events added to the input file and ignores the ones that it has already processed to avoid processing the same event more than once on restart.

    Logstash records its progress in a file that's referred to as a `sincedb` file. Logstash creates a `sincedb` file for each file that it watches for changes.

2. Open the `sincedb` file to check how much of the input files are processed:

    ```bash
    CD数据/插件/输入/文件/
    LS-al

    -RW-r--r--  1个用户工作人员0 6月13日10:50 .sincedb_9e484f2a9e6c0d1bdfe6f23ac107ffc5

    CAT .SINCEDB_9E484F2A9E6C0D1BDFE6F23AC107FFC5

    51575938 1 4 7727
    ```

    The last number in the `sincedb` file (7727) is the byte offset of the last known event processed.

5. To process the input file from the beginning, delete the `sincedb` file:

    ```YML
    rm .sincedb_*
    ```

2. Start Logstash with a ` - -config.reload.automatic` argument:

    ```bash
    bin/logstash-f config/pipeline.conf--config.reload.automatic
    ```

    The `重新加载` option only reloads if you add a new line at the end of the pipeline configuration file.

    Sample output:

    ```YML
    {
       "message" =>"216.243.171.38 - - [20/Sep/2017:19:11:52 +0200] \"get/products/view/123 http/1.1 \" 200 12798 \"https：//codingexplained.com/products \" \"mozilla/5.0（兼容; yandexbot/3.0; +http：//yandex.com/bots）\""，，，，
       "@version" =>"1"，，，，
          "host" =>"a483e711a548.ant.amazon.com"，，，，
          "path" =>"/Users/kumarjao/Desktop/odfe1/logstash-7.12.1/events-data/input_file.log"，，，，
       "@timestamp" => 2021-06-13T18：03：30.423Z
    }
    {
       "message" =>"91.59.108.75 - - [20/Sep/2017:20:11:43 +0200] \"get/js/main.js http/1.1 \" 200 588 \"https：//codingexplained.com/products/view/863 \" \"Mozilla/5.0（Windows NT 6.1; WOW64; RV：45.0）壁虎/20100101 Firefox/45.0 \""，，，，
      "@version" =>"1"，，，，
          "host" =>"a483e711a548.ant.amazon.com"，，，，
          "path" =>"/Users/kumarjao/Desktop/odfe1/logstash-7.12.1/events-data/input_file.log"，，，，
    "@timestamp" => 2021-06-13T18：03：30.424Z
    }
    ```

7. Add a new line to the input file.
    - Logstash immediately detects the change and processes the new line as an event.

8. Make a change to the `pipeline.conf` 文件。
    - LogStash立即检测到更改并重新加载修改后的管道。


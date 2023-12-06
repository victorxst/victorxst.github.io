---
layout: default
title: 拍摄并恢复快照
parent: 快照
nav_order: 10
has_children: false
grand_parent: 可用性和恢复
redirect_from: 
  - /opensearch/snapshots/snapshot-restore/
  - /opensearch/snapshot-restore/
  - /availability-and-recovery/snapshots/snapshot-restore/
---

# 拍摄并恢复快照

快照不是瞬时。他们需要时间完成，并不代表完美的观点-在-集群的时间视图。当快照正在进行中，您仍然可以索引文档并将其他请求发送到集群，但是快照中通常不包含对现有文档的新文档和更新。快照包括OpenSearch启动快照时存在的主要碎片。根据快照线池的大小，在稍微不同的时间中可能包含不同的碎片。

OpenSearch快照是增量的，这意味着它们仅存储自上次成功快照以来已更改的数据。频繁和频繁快照之间的磁盘使用差异通常很小。

换句话说，每小时的快照一周（总共168张快照）可能没有比在一周结束时拍摄单个快照更多的磁盘空间。另外，您拍摄快照的频率越多，完成的时间就越少。一些OpenSearch用户每30分钟一次拍摄快照。

如果您需要删除快照，请确保使用OpenSearch API，而不是导航到存储位置和清除文件。群集的增量快照通常共享许多相同的数据；当您使用API时，OpenSearch仅删除没有其他快照正在使用的数据。
{: .tip }

---

<details closed markdown="block">
  <summary>
    目录
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## 注册存储库

在拍摄快照之前，您必须"register" 快照存储库。快照存储库只是一个存储位置：共享文件系统，亚马逊简单存储服务（Amazon S3），Hadoop分布式文件系统（HDFS）或Azure存储。


### 共享文件系统

1. 要将共享文件系统用作快照存储库，请将其添加到`opensearch.yml`：

   ```yml
   path.repo: ["/mnt/snapshots"]
   ```

   在RPM和Debian安装上，您可以安装文件系统。如果您使用docker install，请在每个节点中添加文件系统`docker-compose.yml` 在开始群集之前：

   ```yml
   volumes:
     - /Users/jdoe/snapshots:/mnt/snapshots
   ```

1. 然后使用REST API注册存储库：

   ```json
   PUT /_snapshot/my-fs-repository
   {
     "type": "fs",
     "settings": {
       "location": "/mnt/snapshots"
     }
   }
   ```
  {% include copy-curl.html %}

You will most likely not need to specify any parameters except for `地点`. For allowed request parameters, see [Register or update snapshot repository API](https://opensearch.org/docs/latest/api-reference/snapshots/create-repository/).

### Amazon S3

1. To use an Amazon S3 bucket as a snapshot repository, install the `存储库-S3` plugin on all nodes:

   ```bash
   sudo ./bin/opensearch-plugin install repository-s3
   ```

   如果您正在使用Docker安装，请参阅[使用插件]({{site.url}}{{site.baseurl}}/opensearch/install/docker#working-with-plugins)。你的`Dockerfile` 应该看起来像这样：

   ```
   来自OpenSearchProject/openSearch：{{site.opensearch_version}}

   ENV AWS_ACCESS_KEY_ID <access-key>
   ENV AWS_SECRET_ACCESS_KEY <secret-key>

   # 选修的
   ENV AWS_SESSION_TOKEN <optional-session-token>

   RUN /usr/share/opensearch/bin/opensearch-plugin install --batch repository-s3
   RUN /usr/share/opensearch/bin/opensearch-keystore create

   RUN echo $AWS_ACCESS_KEY_ID | /usr/share/opensearch/bin/opensearch-keystore add --stdin s3.client.default.access_key
   RUN echo $AWS_SECRET_ACCESS_KEY | /usr/share/opensearch/bin/opensearch-keystore add --stdin s3.client.default.secret_key

   # 选修的
   RUN echo $AWS_SESSION_TOKEN | /usr/share/opensearch/bin/opensearch-keystore add --stdin s3.client.default.session_token
   ```

   After the Docker cluster starts, skip to step 7.

1. Add your AWS access and secret keys to the OpenSearch keystore:

   ```bash
   sudo ./bin/opensearch-keystore add s3.client.default.access_key
   sudo ./bin/opensearch-keystore add s3.client.default.secret_key
   ```

1. (Optional) If you're using temporary credentials, add your session token:

   ```bash
   sudo ./bin/opensearch-keystore add s3.client.default.session_token
   ```

1. (Optional) If you connect to the internet through a proxy, add those credentials:

   ```bash
   sudo ./bin/opensearch-keystore add s3.client.default.proxy.username
   sudo ./bin/opensearch-keystore add s3.client.default.proxy.password
   ```

1. (Optional) Add other settings to `opensearch.yml`:

   ```YML
   s3.client.default.endpoint：s3.amazonaws.com# S3具有其他端点，但是您可能不需要更改此值。
   s3.client.default.max_retries：3# 如果请求失败，则检索数量
   s3.client.default.path_style_access：false# 是否使用弃用路径-样式的水桶URL。
   # 您可能不需要更改此值，但是有关更多信息，请参见https://docs.aws.amazon.com/amazons3/latest/dev/virtualhosting.html#小路-风格-使用权。
   s3.client.default.protocol：https# http或https
   s3.client.default.proxy.host：my-代理人-主持人# 代理服务器的主机名
   s3.client.default.proxy.port：8080# 代理服务器的端口
   s3.client.default.read_timeout：50s# S3连接超时
   s3.client.default.use_throttle_retries：true# 每个连续的重试，客户是否应该等待逐渐等待更长的时间（指数向后）
   S3.Client.default.Region：US-东方-2# AWS区域要使用。对于非-AWS S3存储，需要此值，但没有效果。
   ```

1. （可选）如果您不想使用AWS访问和秘密键，则可以配置S3插件以使用AWS Identity和Access Management（IAM）为服务帐户：

   ```bash
   sudo ./bin/opensearch-keystore add s3.client.default.role_arn
   sudo ./bin/opensearch-keystore add s3.client.default.role_session_name
   ```

   如果您不想配置AWS访问和秘密键，请修改以下内容`opensearch.yml` 环境。确保文件可以通过`repository-s3` 插入：
   
   ```yml
   s3.client.default.identity_token_file: /usr/share/opensearch/plugins/repository-s3/token
   ```

   如果不是复制的选项，则可以在Web身份令牌文件中创建一个符号链接`${OPENSEARCH_PATH_CONFIG}` 文件夹：

   ```
   ln -s $AWS_WEB_IDENTITY_TOKEN_FILE "${OPENSEARCH_PATH_CONFIG}/aws-web-identity-token-file"
   ```

   您可以在以下内容中引用Web身份令牌文件`opensearch.yml` 通过指定针对解决的相对路径的设置`${OPENSEARCH_PATH_CONFIG}`：

   ```yaml
   s3.client.default.identity_token_file: aws-web-identity-token-file
   ```

   IAM角色至少需要上述设置之一。其他设置将从环境变量中获取（如果可用）：`AWS_ROLE_ARN`，`AWS_WEB_IDENTITY_TOKEN_FILE`，`AWS_ROLE_SESSION_NAME`。

1. 如果您更改`opensearch.yml`，您必须重新启动集群中的每个节点。否则，您只需要重新加载安全集群设置：

   ```
   POST /_nodes/reload_secure_settings
   ```
  {% include copy-curl.html %}

1. 如果您还没有一个，则创建一个S3存储桶。要拍摄快照，您需要使用权限才能访问水库。以下IAM政策就是这些权限的一个例子：

   ```json
   {
	   "Version": "2012-10-17",
	   "Statement": [{
		   "Action": [
			   "s3:*"
		   ],
		   "Effect": "Allow",
		   "Resource": [
			   "arn:aws:s3:::your-bucket",
			   "arn:aws:s3:::your-bucket/*"
		   ]
	   }]
   }
   ```

1. Register the repository using the REST API:

   ```json
   PUT /_snapshot/my-s3-repository
   {
     "type": "s3",
     "settings": {
       "bucket": "my-s3-bucket",
       "base_path": "my/snapshot/directory"
     }
   }
   ```
   {% include copy-curl.html %}

You will most likely not need to specify any parameters except for `桶` and `base_path`。有关允许的请求参数，请参阅[注册或更新快照存储库API](https://opensearch.org/docs/latest/api-reference/snapshots/create-repository/)。

## 拍摄快照

创建快照时，您可以指定两种信息：

- 快照存储库的名称
- 快照的名称

以下快照包括所有索引和群集状态：

```json
PUT /_snapshot/my-repository/1
```
{% include copy-curl.html %}

您还可以添加一个请求主体以包括或排除某些索引或指定其他设置：

```json
PUT /_snapshot/my-repository/2
{
  "indices": "opensearch-dashboards*,my-index*,-my-index-2016",
  "ignore_unavailable": true,
  "include_global_state": false,
  "partial": false
}
```
{% include copy-curl.html %}

请求字段| 描述
:--- | :---
`indices` | 您要在快照中包含的索引。您可以使用`,` 要创建索引列表，`*` 指定索引模式，并`-` 排除某些索引。不要在项目之间放置空间。默认为所有索引。
`ignore_unavailable` | 如果是`indices` 列表不存在，是否忽略它而不是失败快照。默认为`false`。
`include_global_state` | 是否将群集状态包括在快照中。默认为`true`。
`partial` | 是否允许部分快照。默认为`false`，如果一个或多个碎片无法存储，则会使整个快照失败。

如果您在采用后立即要求快照，则可能会看到类似的东西：

```json
GET /_snapshot/my-repository/2
{
  "snapshots": [{
    "snapshot": "2",
    "version": "6.5.4",
    "indices": [
      "opensearch_dashboards_sample_data_ecommerce",
      "my-index",
      "opensearch_dashboards_sample_data_logs",
      "opensearch_dashboards_sample_data_flights"
    ],
    "include_global_state": true,
    "state": "IN_PROGRESS",
    ...
  }]
}
```
{% include copy-curl.html %}

请注意，快照仍在进行中。如果您想等待快照在继续之前完成，请添加`wait_for_completion` 您的请求的参数。快照可能需要一段时间才能完成，因此请考虑此选项是否适合您的用例：

```
PUT _snapshot/my-repository/3?wait_for_completion=true
```
{% include copy-curl.html %}

快照具有以下状态：

状态| 描述
:--- | :---
成功| 快照成功地存储了所有碎片。
进行中| 快照当前正在运行。
部分的| 至少一个碎片未能成功存储。仅当您设置时才会发生`partial` 到`true` 拍摄快照时。
失败的| 快照遇到了错误，没有存储任何数据。
不相容| 该快照与该群集上运行的OpenSearch版本不兼容。看[冲突和兼容性](#conflicts-and-compatibility)。

如果当前正在进行中，则不能拍摄快照。检查状态：

```
GET /_snapshot/_status
```
{% include copy-curl.html %}


## 还原快照

恢复快照的第一步是检索现有的快照。查看所有快照存储库：

```
GET /_snapshot/_all
```
{% include copy-curl.html %}

查看存储库中的所有快照：

```
GET /_snapshot/my-repository/_all
```
{% include copy-curl.html %}

然后还原快照：

```
POST /_snapshot/my-repository/2/_restore
```
{% include copy-curl.html %}

就像拍摄快照一样，您可以添加一个请求主体以包括或排除某些索引或指定其他一些设置：

```json
POST /_snapshot/my-repository/2/_restore
{
  "indices": "opensearch-dashboards*,my-index*",
  "ignore_unavailable": true,
  "include_global_state": false,
  "include_aliases": false,
  "partial": false,
  "rename_pattern": "opensearch-dashboards(.+)",
  "rename_replacement": "restored-opensearch-dashboards$1",
  "index_settings": {
    "index.blocks.read_only": false
  },
  "ignore_index_settings": [
    "index.refresh_interval"
  ]
}
```
{% include copy-curl.html %}

请求参数| 描述
:--- | :---
`indices` | 您要还原的索引。您可以使用`,` 要创建索引列表，`*` 指定索引模式，并`-` 排除某些索引。不要在项目之间放置空间。默认为所有索引。
`ignore_unavailable` | 如果是`indices` 列表不存在，是否忽略它而不是使还原操作失败。默认值为false。
`include_global_state` | 是否还原群集状态。默认值为false。
`include_aliases` | 是否要恢复其相关索引的别名。默认是正确的。
`partial` | 是否允许恢复部分快照。默认值为false。
`rename_pattern` | 如果要在还原索引重命名时，请使用此选项指定与要还原的所有索引匹配的正则表达式。使用捕获组（`()`）重复使用索引名称的部分。
`rename_replacement` | 如果要在还原索引重命名索引，请使用此选项指定替换模式。使用`$0` 要包含整个匹配索引名称，`$1` 包括第一个捕获组的内容，依此类推。
`index_settings` | 如果你想改变[索引设置]({{site.url}}{{site.baseurl}}/im-plugin/index-settings/) 在还原操作期间应用，在此处指定它们。你不能改变`index.number_of_shards`。
`ignore_index_settings` | 而不是明确指定新设置`index_settings`，您可以忽略快照中的某些索引设置，并使用还原过程中应用的群集默认设置。你不能忽略`index.number_of_shards`，`index.number_of_replicas`， 或者`index.auto_expand_replicas`。
`storage_type` | `local` 表示所有快照元数据和索引数据将下载到本地存储。<br /> <br>`remote_snapshot` 表示快照元数据将下载到集群中，但远程存储库将仍然是索引数据的权威存储。数据将根据需要下载和缓存以进行服务查询。群集中的至少一个节点必须配置[搜索角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/) 为了使用该类型恢复快照`remote_snapshot`。<br /> <br>默认为`local`。

### 冲突和兼容性

恢复索引时避免命名冲突的一种方法是使用`rename_pattern` 和`rename_replacement` 选项。然后，您可以在必要时使用`_reindex` API结合两者。更简单的方法是在从快照还原之前删除现有索引。

您可以使用`_close` API在从快照恢复之前关闭现有索引，但是快照中的索引必须具有与现有索引相同的碎片数。

我们建议在从快照恢复之前停止写入请求到集群，这有助于避免以下方案：

1. 您删除了一个索引，该索引还删除了它的别名。
1. 对现在的写请求-删除的别名创建了一个与别名相同的名称的新索引。
1. 由于与新索引的命名冲突，快照的别名无法恢复。

快照只是向前-由一个主要版本兼容。如果您有一个旧的快照，有时可以将其还原为中间群集，重新索引所有索引，进行新的快照，然后重复直到您到达所需的版本，但是您可能会发现更容易在手动索引数据中的数据，新集群。

## 安全考虑

如果您使用的是安全插件，快照会有一些其他限制：

- 要执行快照和还原操作，用户必须具有构建的-在`manage_snapshots` 角色。
- 您无法还原包含全球状态或`.opendistro_security` 指数。

如果快照包含全球状态，则必须在执行还原时将其排除。如果您的快照还包含`.opendistro_security` 索引，将其排除或列出您要包含的所有其他索引：

```json
POST /_snapshot/my-repository/3/_restore
{
  "indices": "-.opendistro_security",
  "include_global_state": false
}
```
{% include copy-curl.html %}

这`.opendistro_security` 索引包含敏感数据，因此我们建议您在快照时将其排除。如果您确实需要从快照还原索引，则必须在请求中包含管理证书：

```bash
curl -k --cert ./kirk.pem --key ./kirk-key.pem -XPOST 'https://localhost:9200/_snapshot/my-repository/3/_restore?pretty'
```
{% include copy-curl.html %}

我们强烈建议不要恢复`.opendistro_security` 使用管理证书，因为这样做可以改变整个集群的安全姿势。看[谨慎]({{site.url}}{{site.baseurl}}/security-plugin/configuration/security-admin/#a-word-of-caution) 有关建议的过程来备份和还原安全插件配置。
{: .warning}

## 索引编解码器注意事项

有关索引编解码器的注意事项，请参阅[索引编解码器]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/#snapshots)。


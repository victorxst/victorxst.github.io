---
layout: default
title: 注册快照存储库
parent: 快照API
nav_order: 1
---

# 注册或更新快照存储库
**引入1.0**
{: .label .label-purple }

您可以使用快照API注册一个新的存储库来存储快照或更新现有存储库的信息。

快照存储库有两种类型：

* 文件系统 （`fs`）：有关创建的说明`fs` 存储库，请参阅[注册存储库共享文件系统]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/snapshot-restore/#shared-file-system)。

*亚马逊简单存储服务（Amazon S3）桶（`s3`）：有关创建的说明`s3` 存储库，请参阅[注册存储库Amazon S3]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/snapshot-restore/#amazon-s3)。

有关创建存储库的说明，请参阅[注册存储库]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#register-repository)。

## 路径和HTTP方法

```
POST /_snapshot/my-first-repo/ 
PUT /_snapshot/my-first-repo/
```

## 路径参数

范围| 数据类型| 描述
:--- | :--- | :---
存储库| 细绳| 存储库名称|

## 请求参数

请求参数取决于存储库的类型：`fs` 或者`s3`。

### FS存储库

请求字段| 描述
:--- | :---
`location` | 快照的文件系统目录，例如文件服务器或Samba共享的已安装目录。必须通过所有节点访问。必需的。
`chunk_size` | 在快照操作期间，将大文件分解成块（例如`64mb`，`1gb`），这对于云存储提供商很重要，对于共享文件系统而言，重要的不太重要。默认为`null` （无限）。选修的。
`compress` | 是否压缩元数据文件。此设置不影响可能已压缩的数据文件，具体取决于您的索引设置。默认为`false`。选修的。
`max_restore_bytes_per_sec` | 快照还原的最大速率。默认值为每秒40 MB（`40m`）。选修的。
`max_snapshot_bytes_per_sec` | 快照拍摄的最大速率。默认值为每秒40 MB（`40m`）。选修的。
`remote_store_index_shallow_copy` | 布尔| 确定是否将远程存储索引的快照捕获为浅副本。默认为`false`。
`readonly` | 是否读取存储库-仅有的。从一个群集迁移时有用（`"readonly": false` 注册时）到另一个集群（`"readonly": true` 注册时）。选修的。

#### 示例请求

以下示例寄存`fs` 使用本地目录的存储库`/mnt/snapshots` 作为`location`。

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

#### S3存储库

请求字段| 描述
:--- | :---
`base_path` | 您要存储快照的存储桶中的路径（例如，`my/snapshot/directory`）。选修的。如果未指定，快照存储在S3桶根中。
`bucket` | S3存储桶的名称。必需的。
`buffer_size` | 块超出的阈值`chunk_size`）应分解成碎片（`buffer_size`）并使用其他API发送到S3。默认值是两个值的较小：100 MB或Java堆的5％。有效的值在`5mb` 和`5gb`。我们不建议更改此选项。
`canned_acl` | S3有几个[ACL罐头](https://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl) 那是`repository-s3` 插件可以在S3中创建它们时添加到对象中。默认为`private`。选修的。
`chunk_size` | 在快照操作期间将文件分解成块（例如`64mb`，`1gb`），这对于云存储提供商很重要，对于共享文件系统而言，重要的不太重要。默认为`1gb`。选修的。
`client` | 指定客户端设置时（例如`s3.client.default.access_key`），您可以使用以外的字符串`default` （例如。`s3.client.backup-role.access_key`）。如果使用替代名称，请更改此值以匹配。默认值和建议值为`default`。选修的。
`compress` | 是否压缩元数据文件。此设置不影响可能已压缩的数据文件，具体取决于您的索引设置。默认为`false`。选修的。
`disable_chunked_encoding` | 禁用块编码，以与某些存储服务兼容。默认为`false`。选修的。
`max_restore_bytes_per_sec` | 快照还原的最大速率。默认值为每秒40 MB（`40m`）。选修的。
`max_snapshot_bytes_per_sec` | 快照拍摄的最大速率。默认值为每秒40 MB（`40m`）。选修的。
`readonly` | 是否读取存储库-仅有的。从一个群集迁移时有用（`"readonly": false` 注册时）到另一个集群（`"readonly": true` 注册时）。选修的。
`remote_store_index_shallow_copy` | 布尔| 远程存储索引的快照是否被捕获为浅副本。默认为`false`。
`server_side_encryption` | 是否要在S3存储桶中加密快照文件。此设置使用AES-256与S3-托管密钥。看[使用服务器保护数据-侧加密](https://docs.aws.amazon.com/AmazonS3/latest/dev/serv-side-encryption.html)。默认值为false。选修的。
`storage_class` | 指定[S3存储类](https://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html) 对于快照文件。默认为`standard`。不要使用`glacier` 和`deep_archive` 存储类。选修的。

为了`base_path` 参数，请勿输入`s3://` 输入S3桶详细信息时的前缀。仅需要存储桶的名称。
{:.note}

#### 示例请求

以下请求注册一个新的S3存储库`my-opensearch-repo` 在现有的存储桶中称为`my-open-search-bucket`。默认情况下，所有快照都存储在`my/snapshot/directory`。

```json
PUT /_snapshot/my-opensearch-repo
{
  "type": "s3",
  "settings": {
    "bucket": "my-open-search-bucket",
    "base_path": "my/snapshot/directory"
  }
}
```
{% include copy-curl.html %}

#### 示例响应

成功后，返回以下JSON对象：

```json
{
  "acknowledged": true
}
```

要验证存储库已注册，请使用[获取快照存储库]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot-repository) API，将存储库名称作为`repository` 路径参数。
{:.note}


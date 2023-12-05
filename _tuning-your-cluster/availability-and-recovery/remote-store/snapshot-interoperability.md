---
layout: default
title: 浅快照
nav_order: 15
parent: 远程支持存储
grand_parent: 可用性和恢复
---

# 浅快照

浅复制快照使您可以从整个遥控器参考数据-支持的存储库，而不是在快照存储库中存储来自该段的所有数据。这使得访问段数据比使用普通快照更快，因为段数据未存储在快照存储库中。

## 启用浅拍照

使用[快照API]({{site.url}}{{site.baseurl}}/api-reference/snapshots/create-repository/) 并设置`remote_store_index_shallow_copy` 存储库设置为`true` 为了启用浅快照副本，如下所示：

```bash
PUT /_snapshot/snap_repo
{
        "type": "s3",
        "settings": {
            "bucket": "test-bucket",
            "base_path": "daily-snaps",
            "remote_store_index_shallow_copy": true
        }
    }
```
{％包含副本-curl.html％}

启用后，所有请求都使用[快照API]({{site.url}}{{site.baseurl}}/api-reference/snapshots/index/) 所有快照将保持不变。启用设置后，我们建议不要禁用设置。这样做可能会影响数据耐用性。

## 考虑因素

在使用浅副本快照之前，请考虑以下内容：

- 浅复制快照仅适用于遥控-支持的索引。
- 集群中的所有节点都必须使用OpenSearch 2.10或更高版本来利用浅复制快照。
- 这`incremental` 当前快照和最后一个快照之间的文件计数和大小为`0` 使用浅复制快照时。
- 可搜索的快照在浅层复制快照中不支持。


---
layout: default
title: 模型API
parent: ML Commons API
has_children: true
nav_order: 10
---

# 模型API

ML Commons支持以下模型-级别API：

- [注册型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/register-model/)
- [获取模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/get-model/)
- [部署模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/deploy-model/)
- [Uneploy模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/undeploy-model/)
- [删除模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/delete-model/)

## 模型访问控制注意事项

对于启用模型访问控制的群集，用户可以在具有指定访问级别的模型组中对模型进行API操作，如下所示：

- `public` 模型组：任何用户。
- `restricted` 模型组：只有与模型组共享至少一个后端角色的模型所有者或用户。
- `private` 模型组：仅模型所有者。

对于具有模型访问控制禁用的群集，任何用户都可以在任何模型组中对模型执行API操作。

管理员可以在任何模型组中对模型执行API操作。

有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。


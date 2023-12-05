---
layout: default
title: REST层授权
parent: 访问控制
nav_order: 80
---


# REST层授权

其余层上的授权通过在其余层上提供授权检查机制，为插件和扩展API请求提供了额外的安全性。这种安全级别位于运输层的顶部，并提供了一种互补的授权方法，而无需更换，修改或以任何方式更改运输层上相同过程。最初创建了REST层授权，目的是满足对扩展的授权检查的需求，该扩展名不会在运输层上进行通信。但是，该功能也适用于希望在创建未来插件进行OpenSearch时使用它的开发人员。

对于使用REST层授权工作的用户，分配角色和映射用户和角色的方法以及插件和扩展的一般用法保持不变---唯一的要求是用户熟悉新的权限计划。

另一方面，开发人员将需要了解背后的想法`NamedRoute` 以及如何构建新路线方案。有关详细信息，请参阅[在插件的休息层授权](https://github.com/opensearch-project/security/blob/main/REST_AUTHZ_FOR_PLUGINS.md)。

使用其余层进行授权的好处包括能够在其余层授权请求并过滤未经授权的请求。结果，这减轻了运输层的处理负担，同时允许对访问API的粒状控制。

您必须具有启用安全插件来使用REST层授权。
{： 。笔记 }


## 名为弗洛特

REST层授权使集群管理员能够授予或撤销群集中特定端点的访问。为了实现这一目标，通往资源的路线使用唯一名称。

为了促进休息层授权，OpenSearch项目介绍了[`NamedRoute`](https://github.com/opensearch-project/OpenSearch/blob/main/server/src/main/java/org/opensearch/rest/NamedRoute.java) 用于路线注册。对于开发人员而言，此标准需要一种使用唯一名称的注册路由的新方法。虽然运输操作通常由方法名称，零件和相应的运输操作组成，但此新实现需要方法名称，零件和该路由的唯一名称。顾名思义，至关重要的是，它在所有插件和扩展程序中都是唯一的，换句话说，不注册到任何其他路线。

例如，考虑以下途径以获取异常检测资源：

`_/detectors/<detectorId>/profile`

对于扩展，您可以创建一个`NamedRoute` 从中引用`routeNamePrefix` 价值`settings.yml` 申请资源（`ad` 在这种情况下）并将其添加到路由中以完成唯一名称。结果显示在以下示例中：

`ad:detectors/profile`

对于插件，您可以使用插件的名称而不是`routeNamePrefix` 价值。

然后可以将路由名称映射到一个角色，就像传统权限所绘制的方式一样。以下示例中证明了这一点：

```yml
ad_role:
  reserved: true
  cluster_permissions:
    - 'ad:detectors/profile'
```


## 映射用户和角色

您映射用户和角色的方式没有变化`NamedRoute`。此外，供许可的新格式与现有配置兼容。本节提供了一个示例，说明了用户和角色映射如何出现遗产和`NamedRoute` 配置及其如何授权注册路线进行操作。

当用户启动REST请求时，请检查用户的角色，并评估与用户关联的每个许可，以确定是否与分配给路由的唯一名称或与在此期间定义的任何遗留操作的匹配匹配路线的注册。可以将用户映射到包含针对唯一名称或旧行动的权限的角色。考虑虚构插件的以下角色`abc`：

```yml
abcplugin_read_access:
   reserved: true
   cluster_permissions:
     - 'cluster:admin/opensearch/abcplugin/route/get'
```

还要考虑以下角色映射：

```yml
abcplugin_read_access:
	 reserved: true
	 users:
		 - "user-A"
```

如果`user-A` 将REST API拨打到路线`/_plugins/_abcplugin/route/get`，授予用户授权该操作。但是，对于不同的路线，例如`/_plugins/_abcplugin/route/delete`，请求被拒绝。

相同的逻辑对于使用唯一名称的路线和概念的角色和角色映射也是如此`NamedRoute`。考虑同一插件的以下角色`abc`：

```yml
abcplugin_read_access_nr:
   reserved: true
   cluster_permissions:
     - 'abcplugin:routeGet'
     - 'abcplugin:routePut'
     - 'abcplugin:routeDelete'
```

还要考虑以下角色映射：

```yml
abcplugin_read_access_nr:
	 reserved: true
	 users:
		 - "user-B"
```

在第二种情况下，如果`user-B` 对任何路线进行休息API调用`/_plugins/_abcplugin/route/get`，，，，`/_plugins/_abcplugin/route/put`， 或者`/_plugins/_abcplugin/route/delete`，授予用户授权该操作。



---
layout: default
title: API
parent: 访问控制
nav_order: 120
redirect_from: 
 - /security-plugin/access-control/api/
---

# API

安全插件REST API使您可以通过编程方式创建和管理用户，角色，角色映射，行动组和租户。

---

#### Table of contents
1. TOC
{:toc}


---

## API的访问控制

就像OpenSearch权限一样，您可以使用角色控制对安全插件REST API的访问。指定角色`opensearch.yml`：

```yml
plugins.security.restapi.roles_enabled: ["<role>", ...]
```
{% include copy.html %}

这些角色现在可以访问所有API。为了防止访问某些API：

```yml
plugins.security.restapi.endpoints_disabled.<role>.<endpoint>: ["<method>", ...]
```
{% include copy.html %}

角色还允许您控制对特定REST API的访问。您可以将单个或多个集群权限添加到角色中，并在将其映射到角色时授予用户访问相关的API。以下集群权限列表包括与安全性REST API相对应的端点：

| **允许**                 | **API被授予**                   | **描述**                                                                                                                                    |
|：-------------------------------|：-----------------------------------|：---------------------------------------------------------------------------------------------------------------------------------------------------|
| RESTAPI：管理/行动组| `/actiongroup` 和`/actiongroups` | 获得，删除，创建和修补程序组（包括批量更新）的权限。|
| RESTAPI：管理/允许列表| `/allowlist`                       | 许可将任何端点和HTTP请求添加到允许端点和请求列表中。|
| RESTAPI：管理员/内部使用者| `/internaluser` 和`/user`        | 权限添加，检索，修改和删除集群中的任何用户。|
| RESTAPI：admin/nodesdn| `/nodesdn`                         | 权限添加，检索，更新或删除从允许列表中删除任何明显的名称，并启用集群和/或节点之间的通信。|
| RESTAPI：管理员/角色| `/roles`                           | 许可添加，检索，修改和删除集群中的任何角色。|
| RESTAPI：管理员/角色图| `/rolesmapping`                    | 许可添加，检索，修改和删除任何角色-映射。|
| RESTAPI：admin/ssl/cert/info| `/ssl/certs/info`                  | 允许查看当前运输和HTTP证书。|
| RESTAPI：Admin/SSL/CERTS/RELOOAD| `/ssl/certs/reload`                | 允许查看重新加载运输和HTTP证书。|
| RESTAPI：管理员/租户| `/tenants`                         | 允许获取，删除，创建和补丁租户。|



可能的值`endpoint` 是：

- 行动组
- 角色
- 角色图
- 内部使用者
- config
- 缓存
- 系统信息
- nodesdn
- SSL

可能的值`method` 是：

- 得到
- 放
- 邮政
- 删除
- 修补

例如，以下配置可以授予三个角色访问REST API，但随后防止`test-role` 从制作，发布，删除或补丁请求到`_plugins/_security/api/roles` 或者`_plugins/_security/api/internalusers`：

```yml
plugins.security.restapi.roles_enabled: ["all_access", "security_rest_api_access", "test-role"]
plugins.security.restapi.endpoints_disabled.test-role.ROLES: ["PUT", "POST", "DELETE", "PATCH"]
plugins.security.restapi.endpoints_disabled.test-role.INTERNALUSERS: ["PUT", "POST", "DELETE", "PATCH"]
```
{% include copy.html %}

使用PUT和补丁方法[配置API](#configuration)，将以下行添加到`opensearch.yml`：

```yml
plugins.security.unsupported.restapi.allow_securityconfig_modification: true
```
{% include copy.html %}


## 保留和隐藏的资源

您可以将用户，角色，角色映射和行动组标记为保留。将该标志设置为true的资源无法使用REST API或OPENSEARCH仪表板更改。

要将资源标记为保留，请添加以下标志：

```yml
kibana_user:
  reserved: true
```
{% include copy.html %}

同样，您可以将用户，角色，角色映射和行动组标记为隐藏。将该标志设置为true的资源未由REST API返回，也不可见在OpenSearch仪表板中：

```yml
kibana_user:
  hidden: true
```
{% include copy.html %}

隐藏的资源自动保留。

要添加或删除这些标志，请修改`config/opensearch-security/internal_users.yml` 并运行`plugins/opensearch-security/tools/securityadmin.sh`。


---

## Account

### Get account details
引入1.0
{: .label .label-purple }

返回当前用户的帐户详细信息。例如，如果您将请求签署为`admin` 用户，响应包括该用户的详细信息。


#### Request

```json
获取_plugins/_ security/api/帐户
```
{% include copy-curl.html %}

#### Example response

```json
{
  "user_name"："admin"，
  "is_reserved"： 真的，
  "is_hidden"： 错误的，
  "is_internal_user"： 真的，
  "user_requested_tenant"： 无效的，
  "backend_roles"：[[
    "admin"
  ]，，
  "custom_attribute_names"：[]，，
  "tenants"：{
    "global_tenant"： 真的，
    "admin_tenant"： 真的，
    "admin"： 真的
  }，，
  "roles"：[[
    "all_access"，
    "own_index"
  这是给出的
}
```


### Change password
引入1.0
{: .label .label-purple }

更改当前用户的密码。

#### Path and HTTP methods

```json
放置_plugins/_ security/api/帐户
```
{% include copy-curl.html %}

#### Request fields

| 场地| 数据类型| 描述| 必需的|
|：-------------------|：-----------|：-------------------------------|：----------|
| 当前密码| 细绳| 当前密码。| 是的|
| 密码| 细绳| 要设置的新密码。| 是的|

##### Example request

```json
放置_plugins/_ security/api/帐户
{
    "current_password"："old-password"，
    "password"："new-password"
}
```
{% include copy-curl.html %}


##### Example response

```json
{
  "status"："OK"，
  "message"："'test-user' updated."
}
```

#### Response fields

| 场地| 数据类型| 描述|
|：---------|：-----------|：------------------------------|
| 地位| 细绳| 操作的状态。|
| 信息| 细绳| 描述性消息。|


---

## 行动组

### 获取行动小组
引入1.0
{: .label .label-purple }

检索一个动作小组。

```json
GET _plugins/_security/api/actiongroups/<action-group>
```
{% include copy-curl.html %}

#### 要求

```json
GET _plugins/_security/api/actiongroups/custom_action_group
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "custom_action_group": {
    "reserved": false,
    "hidden": false,
    "allowed_actions": [
      "kibana_all_read",
      "indices:admin/aliases/get",
      "indices:admin/aliases/exists"
    ],
    "description": "My custom action group",
    "static": false
  }
}
```


### 获取行动组
引入1.0
{: .label .label-purple }

检索所有行动组。


#### 要求

```json
GET _plugins/_security/api/actiongroups/
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "read": {
    "reserved": true,
    "hidden": false,
    "allowed_actions": [
      "indices:data/read*",
      "indices:admin/mappings/fields/get*",
      "indices:admin/resolve/index"
    ],
    "type": "index",
    "description": "Allow all read operations",
    "static": true
  },
  "cluster_all": {
    "reserved": true,
    "hidden": false,
    "allowed_actions": [
      "cluster:*"
    ],
    "type": "cluster",
    "description": "Allow everything on cluster level",
    "static": true
  },
  ...
}
```


### 删除行动组
引入1.0
{: .label .label-purple }

#### 要求

```json
DELETE _plugins/_security/api/actiongroups/<action-group>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status":"OK",
  "message":"actiongroup SEARCH deleted."
}
```


### 创建行动组
引入1.0
{: .label .label-purple }

创建或替换指定的操作组。

#### 要求

```json
PUT _plugins/_security/api/actiongroups/<action-group>
{
  "allowed_actions": [
    "indices:data/write/index*",
    "indices:data/write/update*",
    "indices:admin/mapping/put",
    "indices:data/write/bulk*",
    "read",
    "write"
  ]
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "CREATED",
  "message": "'my-action-group' created."
}
```


### 补丁动作组
引入1.0
{: .label .label-purple }

更新操作组的各个属性。

#### 要求

```json
PATCH _plugins/_security/api/actiongroups/<action-group>
[
  {
    "op": "replace", "path": "/allowed_actions", "value": ["indices:admin/create", "indices:admin/mapping/put"]
  }
]
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status":"OK",
  "message":"actiongroup SEARCH deleted."
}
```


### 补丁动作组
引入1.0
{: .label .label-purple }

在单个呼叫中创建，更新或删除多个操作组。

#### 要求

```json
PATCH _plugins/_security/api/actiongroups
[
  {
    "op": "add", "path": "/CREATE_INDEX", "value": { "allowed_actions": ["indices:admin/create", "indices:admin/mapping/put"] }
  },
  {
    "op": "remove", "path": "/CRUD"
  }
]
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status":"OK",
  "message":"actiongroup SEARCH deleted."
}
```


---

## Users

这些调用可让您创建，更新和删除内部用户。如果您使用外部身份验证后端，则可能不必担心内部用户。


### Get user
引入1.0
{: .label .label-purple }

#### Request

```json
获取_plugins/_security/api/internalusers/<username>
```
{% include copy-curl.html %}


#### Example response

```json
{
  "kirk"：{
    "hash"：""，
    "roles"：[["captains"，"starfleet" ]，，
    "attributes"：{
       "attribute1"："value1"，
       "attribute2"："value2"，
    }
  }
}
```


### Get users
引入1.0
{: .label .label-purple }

#### Request

```json
获取_plugins/_ security/api/internestusers/
```
{% include copy-curl.html %}

#### Example response

```json
{
  "kirk"：{
    "hash"：""，
    "roles"：[["captains"，"starfleet" ]，，
    "attributes"：{
       "attribute1"："value1"，
       "attribute2"："value2"，
    }
  }
}
```


### Delete user
引入1.0
{: .label .label-purple }

#### Request

```json
删除_plugins/_security/api/internalusers/<username>
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："user kirk deleted."
}
```


### Create user
引入1.0
{: .label .label-purple }

创建或替换指定的用户。您必须指定`password` （纯文本）或`hash` （哈希用户密码）。如果指定`password`，安全插件在存储密码之前会自动哈希。

请注意，您在`opendistro_security_roles` 安全插件必须已经存在数组才能将用户映射到该角色。要查看预定义的角色，请参阅[预定义角色的列表]（{{site.url}}} {{site.baseurl}}/security/access-控制/用户-角色#预定义-角色）。有关如何创建角色的说明，请参阅[创建角色]（#创造-角色）。

#### Request

```json
放置_plugins/_ security/api/internalusers/<username>
{
  "password"："kirkpass"，
  "opendistro_security_roles"：[["maintenance_staff"，"database_manager"]，，
  "backend_roles"：[["role 1"，"role 2"]，，
  "attributes"：{
    "attribute1"："value1"，
    "attribute2"："value2"
  }
}
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："CREATED"，
  "message"："User kirk created"
}
```


### Patch user
引入1.0
{: .label .label-purple }

更新内部用户的各个属性。

#### Request

```json
补丁_plugins/_ security/api/internalusers/<username>
[
  {
    "op"："replace"，"path"："/backend_roles"，"value"：[["klingons"这是给出的
  }，，
  {
    "op"："replace"，"path"："/opendistro_security_roles"，"value"：[["ship_manager"这是给出的
  }，，
  {
    "op"："replace"，"path"："/attributes"，"value"：{"newattribute"："newvalue" }
  }
这是给出的
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："'kirk' updated."
}
```

### Patch users
引入1.0
{: .label .label-purple }

在单个调用中创建，更新或删除多个内部用户。

#### Request

```json
补丁_plugins/_ security/api/internestusers
[
  {
    "op"："add"，"path"："/spock"，"value"：{"password"："testpassword1"，"backend_roles"：[["testrole1"]}}
  }，，
  {
    "op"："add"，"path"："/worf"，"value"：{"password"："testpassword2"，"backend_roles"：[["testrole2"]}}
  }，，
  {
    "op"："remove"，"path"："/riker"
  }
这是给出的
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："Resource updated."
}
```


---

## 角色


### 发挥作用
引入1.0
{: .label .label-purple }

检索一个角色。

#### 要求

```json
GET _plugins/_security/api/roles/<role>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "test-role": {
    "reserved": false,
    "hidden": false,
    "cluster_permissions": [
      "cluster_composite_ops",
      "indices_monitor"
    ],
    "index_permissions": [{
      "index_patterns": [
        "movies*"
      ],
      "dls": "",
      "fls": [],
      "masked_fields": [],
      "allowed_actions": [
        "read"
      ]
    }],
    "tenant_permissions": [{
      "tenant_patterns": [
        "human_resources"
      ],
      "allowed_actions": [
        "kibana_all_read"
      ]
    }],
    "static": false
  }
}
```


### 获得角色
引入1.0
{: .label .label-purple }

检索所有角色。

#### 要求

```json
GET _plugins/_security/api/roles/
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "manage_snapshots": {
    "reserved": true,
    "hidden": false,
    "description": "Provide the minimum permissions for managing snapshots",
    "cluster_permissions": [
      "manage_snapshots"
    ],
    "index_permissions": [{
      "index_patterns": [
        "*"
      ],
      "fls": [],
      "masked_fields": [],
      "allowed_actions": [
        "indices:data/write/index",
        "indices:admin/create"
      ]
    }],
    "tenant_permissions": [],
    "static": true
  },
  ...
}
```


### 删除角色
引入1.0
{: .label .label-purple }

#### 要求

```json
DELETE _plugins/_security/api/roles/<role>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status":"OK",
  "message":"role test-role deleted."
}
```


### 创建角色
引入1.0
{: .label .label-purple }

创建或替换指定角色。

#### 要求

```json
PUT _plugins/_security/api/roles/<role>
{
  "cluster_permissions": [
    "cluster_composite_ops",
    "indices_monitor"
  ],
  "index_permissions": [{
    "index_patterns": [
      "movies*"
    ],
    "dls": "",
    "fls": [],
    "masked_fields": [],
    "allowed_actions": [
      "read"
    ]
  }],
  "tenant_permissions": [{
    "tenant_patterns": [
      "human_resources"
    ],
    "allowed_actions": [
      "kibana_all_read"
    ]
  }]
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "OK",
  "message": "'test-role' updated."
}
```

>由于与Unicode特殊字符相关的单词边界，Unicode标准分析仪无法索引[文本字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text/) 当包含这些特殊字符之一时，值为整体值。结果，标准分析仪将包含特殊字符的文本字段值解析为由特殊字符隔开的多个值，从而有效地将其两侧的不同元素示意。
>
>例如，由于字段中的值```"user.id": "User-1"``` 和```"user.id": "User-2"``` 包含连字符/减号，此特殊字符将阻止分析仪区分两个不同的用户`user.id` 并将它们解释为同一。这可能导致对文档的无意过滤，并可能损害其访问权限的控制。
>
>为避免这种情况，您可以使用自定义分析仪或将字段映射为`keyword`，执行确切的-匹配搜索。看[关键字字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/keyword/) 对于后一个选项。
>
>对于当字段类型为时应避免的字符列表`text`， 看[单词边界](https://unicode.org/reports/tr29/#Word_Boundaries)。
{: .warning}


### 补丁角色
引入1.0
{: .label .label-purple }

更新角色的个人属性。

#### 要求

```json
PATCH _plugins/_security/api/roles/<role>
[
  {
    "op": "replace", "path": "/index_permissions/0/fls", "value": ["myfield1", "myfield2"]
  },
  {
    "op": "remove", "path": "/index_permissions/0/dls"
  }
]
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "OK",
  "message": "'<role>' updated."
}
```


### 补丁角色
引入1.0
{: .label .label-purple }

在单个呼叫中创建，更新或删除多个角色。

#### 要求

```json
PATCH _plugins/_security/api/roles
[
  {
    "op": "replace", "path": "/role1/index_permissions/0/fls", "value": ["test1", "test2"]
  },
  {
    "op": "remove", "path": "/role1/index_permissions/0/dls"
  },
  {
    "op": "add", "path": "/role2/cluster_permissions", "value": ["manage_snapshots"]
  }
]
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "OK",
  "message": "Resource updated."
}
```


---

## Role mappings

### Get role mapping
引入1.0
{: .label .label-purple }

检索一个角色映射。

#### Request

```json
获取_plugins/_ security/api/rolesmapping/<角色>
```
{% include copy-curl.html %}

#### Example response

```json
{
  "role_starfleet" ：{
    "backend_roles" ：[["starfleet"，"captains"，"defectors"，"cn=ldaprole,ou=groups,dc=example,dc=com" ]，，
    "hosts" ：[["*.starfleetintranet.com" ]，，
    "users" ：[["worf" 这是给出的
  }
}
```


### Get role mappings
引入1.0
{: .label .label-purple }

检索所有角色映射。

#### Request

```json
获取_plugins/_ security/api/rolesmapping
```
{% include copy-curl.html %}

#### Example response

```json
{
  "role_starfleet" ：{
    "backend_roles" ：[["starfleet"，"captains"，"defectors"，"cn=ldaprole,ou=groups,dc=example,dc=com" ]，，
    "hosts" ：[["*.starfleetintranet.com" ]，，
    "users" ：[["worf" 这是给出的
  }
}
```


### Delete role mapping
引入1.0
{: .label .label-purple }

删除指定的角色映射。

#### Request

```json
delete _plugins/_security/api/rolesmapping/<croun>
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："'my-role' deleted."
}
```


### Create role mapping
引入1.0
{: .label .label-purple }

创建或替换指定的角色映射。

#### Request

```json
放置_plugins/_ security/api/rolesmapping/<croun>
{
  "backend_roles" ：[["starfleet"，"captains"，"defectors"，"cn=ldaprole,ou=groups,dc=example,dc=com" ]，，
  "hosts" ：[["*.starfleetintranet.com" ]，，
  "users" ：[["worf" 这是给出的
}
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："CREATED"，
  "message"："'my-role' created."
}
```


### Patch role mapping
引入1.0
{: .label .label-purple }

更新角色映射的各个属性。

#### Request

```json
补丁_plugins/_ security/api/rolesmapping/<croum>
[
  {
    "op"："replace"，"path"："/users"，"value"：[["myuser"这是给出的
  }，，
  {
    "op"："replace"，"path"："/backend_roles"，"value"：[["mybackendrole"这是给出的
  }
这是给出的
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："'my-role' updated."
}
```


### Patch role mappings
引入1.0
{: .label .label-purple }

在单个呼叫中创建或更新多个角色映射。

#### Request

```json
补丁_plugins/_ security/api/rolesmapping
[
  {
    "op"："add"，"path"："/human_resources"，"value"：{"users"：[["user1"]，，"backend_roles"：[["backendrole2"]}}
  }，，
  {
    "op"："add"，"path"："/finance"，"value"：{"users"：[["user2"]，，"backend_roles"：[["backendrole2"]}}
  }
这是给出的
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："Resource updated."
}
```


---

## 租户

### 找租户
引入1.0
{: .label .label-purple }

检索一个租户。

#### 要求

```json
GET _plugins/_security/api/tenants/<tenant>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "human_resources": {
    "reserved": false,
    "hidden": false,
    "description": "A tenant for the human resources team.",
    "static": false
  }
}
```


### 找租户
引入1.0
{: .label .label-purple }

检索所有租户。

#### 要求

```json
GET _plugins/_security/api/tenants/
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "global_tenant": {
    "reserved": true,
    "hidden": false,
    "description": "Global tenant",
    "static": true
  },
  "human_resources": {
    "reserved": false,
    "hidden": false,
    "description": "A tenant for the human resources team.",
    "static": false
  }
}
```


### 删除房客
引入1.0
{: .label .label-purple }

删除指定的租户。

#### 要求

```json
DELETE _plugins/_security/api/tenants/<tenant>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status":"OK",
  "message":"tenant human_resources deleted."
}
```


### 创建房客
引入1.0
{: .label .label-purple }

创建或替换指定的租户。

#### 要求

```json
PUT _plugins/_security/api/tenants/<tenant>
{
  "description": "A tenant for the human resources team."
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status":"CREATED",
  "message":"tenant human_resources created"
}
```


### 补丁租户
引入1.0
{: .label .label-purple }

添加，删除或修改单个租户。

#### 要求

```json
PATCH _plugins/_security/api/tenants/<tenant>
[
  {
    "op": "replace", "path": "/description", "value": "An updated description"
  }
]
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "OK",
  "message": "Resource updated."
}
```


### 补丁租户
引入1.0
{: .label .label-purple }

在一个呼叫中添加，删除或修改多个租户。

#### 要求

```json
PATCH _plugins/_security/api/tenants/
[
  {
    "op": "replace",
    "path": "/human_resources/description",
    "value": "An updated description"
  },
  {
    "op": "add",
    "path": "/another_tenant",
    "value": {
      "description": "Another description."
    }
  }
]
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "OK",
  "message": "Resource updated."
}
```

---


## Configuration

### Get configuration
引入1.0
{: .label .label-purple }

以JSON格式检索当前的安全插件配置。

#### Request

```json
获取_plugins/_ security/api/securityConfig
```
{% include copy-curl.html %}


### Update configuration
引入1.0
{: .label .label-purple }

使用REST API创建或更新现有配置。此操作可以轻松破坏您现有的配置，因此我们建议使用`securityadmin.sh` 相反，这更安全。请参阅[API的访问控制]（#使用权-控制-为了-这-API）如何启用此操作。

#### Request

```json
放置_plugins/_ security/api/securityConfig/config
{
  "dynamic"：{
    "filtered_alias_mode"："warn"，
    "disable_rest_auth"： 错误的，
    "disable_intertransport_auth"： 错误的，
    "respect_request_indices_options"： 错误的，
    "opensearch-dashboards"：{
      "multitenancy_enabled"： 真的，
      "server_username"："kibanaserver"，
      "index"：".opensearch-dashboards"
    }，，
    "http"：{
      "anonymous_auth_enabled"： 错误的
    }，，
    "authc"：{
      "basic_internal_auth_domain"：{
        "http_enabled"： 真的，
        "transport_enabled"： 真的，
        "order"：0，
        "http_authenticator"：{
          "challenge"： 真的，
          "type"："basic"，
          "config"：{}
        }，，
        "authentication_backend"：{
          "type"："intern"，
          "config"：{}
        }，，
        "description"："Authenticate via HTTP Basic against internal users database"
      }
    }，，
    "auth_failure_listeners"：{}，
    "do_not_fail_on_forbidden"： 错误的，
    "multi_rolespan_enabled"： 真的，
    "hosts_resolver_mode"："ip-only"，
    "do_not_fail_on_forbidden_empty"： 错误的
  }
}
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："'config' updated."
}
```


### Patch configuration
引入1.0
{: .label .label-purple }

使用REST API更新现有配置。此操作可以轻松破坏您现有的配置，因此我们建议使用`securityadmin.sh` 相反，这更安全。请参阅[API的访问控制]（#使用权-控制-为了-这-API）如何启用此操作。

在执行操作之前，必须首先将以下行添加到`opensearch.yml`：

```yml
插件
```
{% include copy.html %}

#### Request

```json
补丁_plugins/_ security/api/securityConfig
[
  {
    "op"："replace"，"path"："/config/dynamic/authc/basic_internal_auth_domain/transport_enabled"，"value"："true"
  }
这是给出的
```
{% include copy-curl.html %}

#### Example response

```json
{
  "status"："OK"，
  "message"："Resource updated."
}
```

---

## 杰出的名字

这些REST API让Super Admin（或具有足够权限访问此API的用户）添加，检索，更新或从允许列表中删除任何杰出名称，以启用群集和/或节点之间的通信。

在使用REST API配置允许列表之前，您必须首先将以下行添加到`opensearch.yml`：

```yml
plugins.security.nodes_dn_dynamic_config_enabled: true
```
{% include copy.html %}


### 获取杰出的名字

检索允许列表中的所有杰出名称。

#### 要求

```json
GET _plugins/_security/api/nodesdn
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "cluster1": {
    "nodes_dn": [
      "CN=cluster1.example.com"
    ]
  }
}
```

要从特定集群或节点的允许列表中获取杰出的名称，请在请求路径中包含群集的名称。

#### 要求

```json
GET _plugins/_security/api/nodesdn/<cluster-name>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "cluster3": {
    "nodes_dn": [
      "CN=cluster3.example.com"
    ]
  }
}
```


### 更新杰出的名称

添加或更新集群或节点的允许列表中指定的杰出名称。

#### 要求

```json
PUT _plugins/_security/api/nodesdn/<cluster-name>
{
  "nodes_dn": [
    "CN=cluster3.example.com"
  ]
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "CREATED",
  "message": "'cluster3' created."
}
```

### 更新所有杰出名称

对杰出名称列表进行批量更新。

#### 路径和HTTP方法

```json
PATCH _plugins/_security/api/nodesdn
```
{% include copy-curl.html %}

#### 请求字段

| 场地| 数据类型| 描述| 必需的|
|：----------------|：-----------|：------------------------------------------------------------------------------------------------------------------|：---------|
| OP| 细绳| 在动作组上执行的操作。可能的值：`remove`，`add`，`replace`，`move`，`copy`，`test`。| 是的|
| 小路| 细绳| 资源的路径。| 是的|
| 价值| 大批| 用于更新的新值。| 是的|


##### 示例请求

```
PATCH _plugins/_security/api/nodesdn
[
   {
      "op":"replace",
      "path":"/cluster1/nodes_dn/0",
      "value": ["CN=Karen Berge,CN=admin,DC=corp,DC=Fabrikam,DC=COM", "CN=George Wall,CN=admin,DC=corp,DC=Fabrikam,DC=COM"]
   }
]
```
{% include copy-curl.html %}

##### 示例响应

```json
{
  "status":"OK",
  "message":"Resources updated."
}
```

#### 响应字段

| 场地| 数据类型| 描述|
|：--------|：----------|：---------------------|
| 地位| 细绳| 响应状态。|
| 信息| 细绳| 响应消息。|


### 删除杰出的名称

删除指定集群或节点的允许列表中的所有区分名称。

#### 要求

```json
DELETE _plugins/_security/api/nodesdn/<cluster-name>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status": "OK",
  "message": "'cluster3' deleted."
}
```


---

## Certificates

### Get certificates
引入1.0
{: .label .label-purple }

检索集群的安全证书。

#### Request

```json
获取_plugins/_security/api/ssl/certs
```
{% include copy-curl.html %}

#### Example response

```json
{
  "http_certificates_list"：[[
    {
      "issuer_dn"："CN=Example Com Inc. Root CA,OU=Example Com Inc. Root CA,O=Example Com Inc.,DC=example,DC=com"，
      "subject_dn"："CN=node-0.example.com,OU=node,O=node,L=test,DC=de"，
      "san"："[[8, 1.2.3.4.5.5], [2, node-0.example.com]"，
      "not_before"："2018-04-22T03:43:47Z"，
      "not_after"："2028-04-19T03:43:47Z"
    }
  ]，，
  "transport_certificates_list"：[[
    {
      "issuer_dn"："CN=Example Com Inc. Root CA,OU=Example Com Inc. Root CA,O=Example Com Inc.,DC=example,DC=com"，
      "subject_dn"："CN=node-0.example.com,OU=node,O=node,L=test,DC=de"，
      "san"："[[8, 1.2.3.4.5.5], [2, node-0.example.com]"，
      "not_before"："2018-04-22T03:43:47Z"，
      "not_after"："2028-04-19T03:43:47Z"
    }
  这是给出的
}
```

### Reload transport certificates

重新加载传输层通信证书。这些REST API让超级管理员（或具有足够权限的用户访问此API）重新加载传输层证书。

#### Path and HTTP methods

```json
put/_plugins/_security/api/ssl/ssl/transport/reloadcerts
```
{% include copy-curl.html %}

##### Example request

```bash
卷曲-x放"https://your-opensearch-cluster/_plugins/_security/api/ssl/transport/reloadcerts"
```
{% include copy-curl.html %}

##### Example response

```json
{
  "status"："OK"，
  "message"："updated transport certs"
}
```

#### Response fields

| 场地| 数据类型| 描述|
|：--------|：----------|：----------------------------------------------------------------------------------|
| 地位| 细绳| 表示操作的状态。可能的值："OK" 或错误消息。|
| 信息| 细绳| 有关操作的其他信息。|


#### Reload HTTP certificates

重新加载HTTP层通信证书。这些REST API让超级管理员（或具有足够权限的用户访问此API）重新加载HTTP层证书。

#### Path and HTTP methods

```json
put/_plugins/_security/api/ssl/http/reloadcerts
```
{% include copy-curl.html %}


##### Example request

```
卷曲-x放"https://your-opensearch-cluster/_plugins/_security/api/ssl/http/reloadcerts"
```
{% include copy-curl.html %}

##### Example response

```json
{
  "status"："OK"，
  "message"："updated http certs"
}
```

#### Response fields

| 场地| 数据类型| 描述|
|：--------|：----------|：--------------------------------------------------------------------|
| 地位| 细绳| API操作的状态。可能的值："OK"。|
| 信息| 细绳| 一条表示HTTP证书已更新的消息。|

---

## 缓存

### 冲洗缓存
引入1.0
{: .label .label-purple }

冲洗安全插件用户，身份验证和授权缓存。


#### 要求

```json
DELETE _plugins/_security/api/cache
```
{% include copy-curl.html %}


#### 示例响应

```json
{
  "status": "OK",
  "message": "Cache flushed successfully."
}
```


---

## Health

### Health check
引入1.0
{: .label .label-purple }

检查安全插件是否启动并运行。如果您在负载平衡器后面操作群集，则此操作对于确定节点健康非常有用，并且不需要签名请求。


#### Request

```json
获取_plugins/_ security/health
```
{% include copy-curl.html %}


#### Example response

```json
{
  "message"： 无效的，
  "mode"："strict"，
  "status"："UP"
}
```


---

## 审核日志

以下API可在安全插件中审核记录。

### 启用审核日志

此API允许您启用或禁用审核日志记录，定义用于审核日志记录和合规性的配置，并对设置进行更新。

有关使用审核日志记录以跟踪访问OpenSearch群集的详细信息，以及有关进一步配置的信息，请参阅[审核日志]({{site.url}}{{site.baseurl}}/security/audit-logs/index/)。

您可以对审核记录进行初始配置`audit.yml` 文件，在`opensearch-project/security/config` 目录。此后，您可以使用REST API或仪表板进行进一步更改配置。
{： 笔记。}

#### 请求字段

场地| 数据类型| 描述
:--- | :--- | :---
`enabled` | 布尔| 启用或禁用审核记录。默认为`true`。
`audit` | 目的| 包含用于审核记录配置的字段。
`audit.ignore_users` | 大批| 用户将被排除在审核之外。支持通配符模式<br>示例：`ignore_users: ["test-user", employee-*"]`
`audit.ignore_requests` | 大批| 要求被排除在审核之外。支持通配符模式。<br>示例：`ignore_requests: ["indices:data/read/*", "SearchRequest"]`
`audit.disabled_rest_categories` | 大批| 将排除REST API审核的类别。默认类别是`AUTHENTICATED`，`GRANTED_PRIVILEGES`。
`audit.disabled_transport_categories` | 大批| 将排除运输API审核的类别。默认类别是`AUTHENTICATED`，`GRANTED_PRIVILEGES`。
`audit.log_request_body` | 布尔| 包括休息和运输层的请求主体（如果有）。默认为`true`。
`audit.resolve_indices` | 布尔| 记录所有受请求影响的索引。解决别名和通配符/日期模式。默认为`true`。
`audit.resolve_bulk_requests` | 布尔| 根据批量请求记录单个操作。默认为`false`。
`audit.exclude_sensitive_headers` | 布尔| 不包括敏感的标头不包括在日志中。默认为`true`。
`audit.enable_transport` | 布尔| 启用/禁用运输API审核。默认为`true`。
`audit.enable_rest` | 布尔| 启用/禁用REST API审核。默认为`true`。
`compliance` | 目的| 包含用于合规配置的字段。
`compliance.enabled` | 布尔| 启用或禁用合规性。默认为`true`。
`compliance.write_log_diffs` | 布尔| 日志仅用于文档更新的差异。默认为`false`。
`compliance.read_watched_fields` | 目的| 索引和字段的地图，以监视读取事件。索引名称和字段都支持通配符模式。
`compliance.read_ignore_users` | 大批| 读取事件的用户列表。支持通配符模式。<br>示例：`read_ignore_users: ["test-user", "employee-*"]`
`compliance.write_watched_indices` | 大批| 索引列表要查看写入事件。支持通配符模式。<br>示例：`write_watched_indices: ["twitter", "logs-*"]`
`compliance.write_ignore_users` | 大批| 为写事件而忽略的用户列表。支持通配符模式。<br>示例：`write_ignore_users: ["test-user", "employee-*"]`
`compliance.read_metadata_only` | 布尔| 仅记录文档的读取事件的元数据。默认为`true`。
`compliance.write_metadata_only` | 布尔| 仅记录文档的写入事件的元数据。默认为`true`。
`compliance.external_config` | 布尔| 记录节点的外部配置文件。默认为`false`。
`compliance.internal_config` | 布尔| 日志更新内部安全性更改。默认为`true`。

更改`_readonly` 属性导致409误差，如下响应中所示。
{: .note}

```json
{
  "status" : "error",
  "reason" : "Invalid configuration",
  "invalid_keys" : {
    "keys" : "_readonly,config"
  }
}
```

#### 示例请求

**得到**

ag呼叫检索审核配置。

```json
GET /_opendistro/_security/api/audit
```
{% include copy-curl.html %}

**放**

put呼叫更新审核配置。

```json
PUT /_opendistro/_security/api/audit/config
{
  "enabled": true,
  "audit": {
    "ignore_users": [],
    "ignore_requests": [],
    "disabled_rest_categories": [
      "AUTHENTICATED",
      "GRANTED_PRIVILEGES"
    ],
    "disabled_transport_categories": [
      "AUTHENTICATED",
      "GRANTED_PRIVILEGES"
    ],
    "log_request_body": false,
    "resolve_indices": false,
    "resolve_bulk_requests": false,
    "exclude_sensitive_headers": true,
    "enable_transport": false,
    "enable_rest": true
  },
  "compliance": {
    "enabled": true,
    "write_log_diffs": false,
    "read_watched_fields": {},
    "read_ignore_users": [],
    "write_watched_indices": [],
    "write_ignore_users": [],
    "read_metadata_only": true,
    "write_metadata_only": true,
    "external_config": false,
    "internal_config": true
  }
}
```
{% include copy-curl.html %}

**修补**

补丁调用用于更新审核配置中的指定字段。补丁方法需要操作，路径和值才能完成有效的请求。有关使用补丁方法的详细信息，请参见以下[修补资源](https://en.wikipedia.org/wiki/PATCH_%28HTTP%29#Patching_resources) Wikipedia的描述。

使用补丁方法还要求用户具有包含管理证书加密的安全配置。要了解有关这些证书的更多信息，请参阅[配置管理证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/#configuring-admin-certificates)。

```bash
curl -X PATCH -k -i --cert <admin_cert file name> --key <admin_cert_key file name> <domain>/_opendistro/_security/api/audit -H 'Content-Type: application/json' -d'[{"op":"add","path":"/config/enabled","value":"true"}]'
```
{% include copy.html %}

OpenSearch仪表板开发工具当前不支持补丁方法。您可以使用[卷曲](https://curl.se/)，[邮差](https://www.postman.com/)，或使用此方法更新配置的另一个替代过程。要遵循GitHub问题以支持仪表板中的补丁方法，请参见[问题#2343](https://github.com/opensearch-project/OpenSearch-Dashboards/issues/2343)。
{: .note}

#### 示例响应

Get Call产生的响应似乎类似于以下内容：

```json
{
  "_readonly" : [
    "/audit/exclude_sensitive_headers",
    "/compliance/internal_config",
    "/compliance/external_config"
  ],
  "config" : {
    "compliance" : {
      "enabled" : true,
      "write_log_diffs" : false,
      "read_watched_fields" : { },
      "read_ignore_users" : [ ],
      "write_watched_indices" : [ ],
      "write_ignore_users" : [ ],
      "read_metadata_only" : true,
      "write_metadata_only" : true,
      "external_config" : false,
      "internal_config" : true
    },
    "enabled" : true,
    "audit" : {
      "ignore_users" : [ ],
      "ignore_requests" : [ ],
      "disabled_rest_categories" : [
        "AUTHENTICATED",
        "GRANTED_PRIVILEGES"
      ],
      "disabled_transport_categories" : [
        "AUTHENTICATED",
        "GRANTED_PRIVILEGES"
      ],
      "log_request_body" : true,
      "resolve_indices" : true,
      "resolve_bulk_requests" : true,
      "exclude_sensitive_headers" : true,
      "enable_transport" : true,
      "enable_rest" : true
    }
  }
}
```
观点请求产生的响应似乎类似于以下内容：

```json
{
  "status" : "OK",
  "message" : "'config' updated."
}
```

补丁请求产生类似于以下响应：

```bash
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 45
```


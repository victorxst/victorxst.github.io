---
layout: default
title: 映射API
parent: API工具
nav_order: 45
---

# 映射API

以下API可用于与映射有关的许多任务，从创建到获取和更新映射。

---
## Get Mappings View

此API返回用作日志源的索引中包含的字段的视图。

### Request fields

以下字段用于获取字段映射。

场地| 类型| 描述
:--- | :--- |：--- 
`index_name` | 细绳| 用于日志摄入的索引的名称。
`rule_topic` | 细绳| 索引的日志类型。

#### Example request

```json
get/_plugins/_security_analytics/映射/查看

{
   "index_name"："windows"，
   "rule_topic"："windows"
}
```

#### Example response

```json
{
    "properties"：{
        "windows-event_data-CommandLine"：{
            "path"："CommandLine"，
            "type"："alias"
        }，，
        "event_uid"：{
            "path"："EventID"，
            "type"："alias"
        }
    }，，
    "unmapped_index_fields"：[[
        "windows-event_data-CommandLine"，
        "unmapped_HiveName"，
        "src_ip"，
        "sha1"，
        "processPath"，
        "CallerProcessName"，
        "CallTrace"，
        "AuthenticationPackageName"，
        "AuditSourceName"，
        "AuditPolicyChanges"，
        "AttributeValue"，
        "AttributeLDAPDisplayName"，
        "ApplicationPath"，
        "Application"，
        "AllowedToDelegateTo"，
        "Address"，
        "Action"，
        "AccountType"，
        "AccountName"，
        "Accesses"，
        "AccessMask"，
        "AccessList"
    这是给出的
}
```

---
## 创建映射

#### 示例请求

```json
POST /_plugins/_security_analytics/mappings

{
   "index_name": "windows",
   "rule_topic": "windows",
   "partial": true,
   "alias_mappings": {
        "properties": {
            "event_uid": {
            "type": "alias",
            "path": "EventID"
          }
       }
   }
}
```

#### 示例响应

```json
{
    "acknowledged": true
}
```

---
## Get Mappings

#### Example request

```json
获取/_plugins/_security_analytics/映射
```

#### Example response

```json
{
    "windows"：{
        "mappings"：{
            "properties"：{
                "windows-event_data-CommandLine"：{
                    "type"："alias"，
                    "path"："CommandLine"
                }，，
                "event_uid"：{
                    "type"："alias"，
                    "path"："EventID"
                }
            }
        }
    }
}
```

---
## 更新映射

#### 示例请求

```json
PUT /_plugins/_security_analytics/mappings

{
   "index_name": "windows",
   "field": "CommandLine",
   "alias": "windows-event_data-CommandLine"
}
```

#### 示例响应

```json
{
    "acknowledged": true
}
```



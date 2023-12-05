---
layout: default
title: 规则API
parent: API工具
nav_order: 40
---

# 规则API

以下API可用于与规则相关的许多任务，从搜索PRE-包装规则以创建和更新自定义规则。

---
## Create Custom Rule

创建自定义规则API使用Sigma安全规则格式来创建自定义规则。有关如何以Sigma格式编写规则的信息，请参见[Sigma的GitHub存储库]（https://github.com/sigmahq/sigma）提供的信息。

```json
post/_plugins/_security_analytics/规则？category = Windows
```

### Example request

```yml
标头：
内容-类型：应用程序/JSON

身体：

title: Moriya rootkit
ID：25B9C01C-350d-4B95-床1-836D04A4F324
description: 如Securelist的操作TunnelsNake报告中所述，检测使用Moriya rootkit
状态：实验
作者：Bhabesh Raj
date: 2021/05/06
修改：2021/11/30
参考：
    - https://securelist.com/operation-Tunnelsnake-和-莫里亚-rootkit/101831
tags: [""]
    - 攻击
    - Attack.privilege_escalation
    - Attack.T1543.003
logsource：
    产品：Windows
    服务：系统
检测：
    选择：
        provider_name：“服务控制经理”
        EventID：7045
        SERVICENAME：ZZNETSVC
    条件：选择
级别：关键
误报：
    - 未知
```

### Example response

**样本1：**

```json
{
    "_id"："M1Rm1IMByX0LvTiGvde2"，
    "_version"：1，
    "rule"：{
        "category"："windows"，
        "title"："Moriya Rootkit"，
        "log_source"：""，
        "description"："Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report"，
        "tags"：[[
            {
                "value"："attack.persistence"
            }，，
            {
                "value"："attack.privilege_escalation"
            }，，
            {
                "value"："attack.t1543.003"
            }
        ]，，
        "references"：[[
            {
                "value"："https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831"
            }
        ]，，
        "level"："critical"，
        "false_positives"：[[
            {
                "value"："Unknown"
            }
        ]，，
        "author"："Bhabesh Raj"，
        "status"："experimental"，
        "last_update_time"："2021-05-06T00:00:00.000Z"，
        "rule"："title: Moriya Rootkit\nid: 25b9c01c-350d-4b95-bed1-836d04a4f324\ndescription: Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report\nstatus: experimental\nauthor: Bhabesh Raj\ndate: 2021/05/06\nmodified: 2021/11/30\nreferences:\n    - https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831\ntags:\n    - attack.persistence\n    - attack.privilege_escalation\n    - attack.t1543.003\nlogsource:\n    product: windows\n    service: system\ndetection:\n    selection:\n        Provider_Name: 'Service Control Manager'\n        EventID: 7045\n        ServiceName: ZzNetSvc\n    condition: selection\nlevel: critical\nfalsepositives:\n    - Unknown"
    }
}
```

**样本2：**

```json
{
  "error"：{
    "root_cause"：[[
      {
        "type"："security_analytics_exception"，
        "reason"："{\"错误\":\"Sigma规则必须具有日志源\",\"错误\":\"Sigma规则必须具有检测定义\"}"
      }
    ]，，
    "type"："security_analytics_exception"，
    "reason"："{\"错误\":\"Sigma规则必须具有日志源\",\"错误\":\"Sigma规则必须具有检测定义\"}"，
    "caused_by"：{
      "type"："exception"，
      "reason"："java.util.Arrays$ArrayList: {\"错误\":\"Sigma规则必须具有日志源\",\"错误\":\"Sigma规则必须具有检测定义\"}"
    }
  }，，
  "status"：400
}
```

---
## 更新自定义规则（不强制）

### 示例请求

```json
PUT /_plugins/_security_analytics/rules/ZaFv1IMBdLpXWBiBa1XI?category=windows

Content-Type: application/json

Body:

title: Moriya Rooskit
id: 25b9c01c-350d-4b95-bed1-836d04a4f324
description: Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report
status: experimental
author: Bhabesh Raj
date: 2021/05/06
modified: 2021/11/30
references:
    - https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831
tags:
    - attack.persistence
    - attack.privilege_escalation
    - attack.t1543.003
logsource:
    product: windows
    service: system
detection:
    selection:
        Provider_Name: 'Service Control Manager'
        EventID: 7045
        ServiceName: ZzNetSvc
    condition: selection
level: critical
falsepositives:
    - Unknown
```

### 示例响应

```json
{
    "error": {
        "root_cause": [
            {
                "type": "security_analytics_exception",
                "reason": "Rule with id ZaFv1IMBdLpXWBiBa1XI is actively used by detectors. Update can be forced by setting forced flag to true"
            }
        ],
        "type": "security_analytics_exception",
        "reason": "Rule with id ZaFv1IMBdLpXWBiBa1XI is actively used by detectors. Update can be forced by setting forced flag to true",
        "caused_by": {
            "type": "exception",
            "reason": "org.opensearch.OpenSearchStatusException: Rule with id ZaFv1IMBdLpXWBiBa1XI is actively used by detectors. Update can be forced by setting forced flag to true"
        }
    },
    "status": 500
}
```

---
## Update Custom Rule (forced)

### Example request

```json
put/_plugins/_security_analytics/ulues/zafv1imbdlpxwbiba1xi？

内容-类型：应用程序/JSON

身体：

title: Moriya Rooskit
ID：25B9C01C-350d-4B95-床1-836D04A4F324
description: 如Securelist的操作TunnelsNake报告中所述，检测使用Moriya rootkit
状态：实验
作者：Bhabesh Raj
date: 2021/05/06
修改：2021/11/30
参考：
    - https://securelist.com/operation-Tunnelsnake-和-莫里亚-rootkit/101831
tags: [""]
    - 攻击
    - Attack.privilege_escalation
    - Attack.T1543.003
logsource：
    产品：Windows
    服务：系统
检测：
    选择：
        provider_name：“服务控制经理”
        EventID：7045
        SERVICENAME：ZZNETSVC
    条件：选择
级别：关键
误报：
    - 未知
```

### Example response

```json
{
    "_id"："ZaFv1IMBdLpXWBiBa1XI"，
    "_version"：1，
    "rule"：{
        "category"："windows"，
        "title"："Moriya Rooskit"，
        "log_source"：""，
        "description"："Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report"，
        "tags"：[[
            {
                "value"："attack.persistence"
            }，，
            {
                "value"："attack.privilege_escalation"
            }，，
            {
                "value"："attack.t1543.003"
            }
        ]，，
        "references"：[[
            {
                "value"："https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831"
            }
        ]，，
        "level"："critical"，
        "false_positives"：[[
            {
                "value"："Unknown"
            }
        ]，，
        "author"："Bhabesh Raj"，
        "status"："experimental"，
        "last_update_time"："2021-05-06T00:00:00.000Z"，
        "rule"："title: Moriya Rooskit\nid: 25b9c01c-350d-4b95-bed1-836d04a4f324\ndescription: Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report\nstatus: experimental\nauthor: Bhabesh Raj\ndate: 2021/05/06\nmodified: 2021/11/30\nreferences:\n    - https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831\ntags:\n    - attack.persistence\n    - attack.privilege_escalation\n    - attack.t1543.003\nlogsource:\n    product: windows\n    service: system\ndetection:\n    selection:\n        Provider_Name: 'Service Control Manager'\n        EventID: 7045\n        ServiceName: ZzNetSvc\n    condition: selection\nlevel: critical\nfalsepositives:\n    - Unknown"
    }
}
```

---
## 搜索pre-包装规则

### 示例请求

```json
POST /_plugins/_security_analytics/rules/_search?pre_packaged=true

{
  "from": 0,
  "size": 20,  
  "query": {
    "nested": {
      "path": "rule",
      "query": {
        "bool": {
          "must": [
            { "match": { "rule.category": "windows" } }
          ]
        }
      }
    }
  }
}
```

### 示例响应

```json
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1580,
            "relation": "eq"
        },
        "max_score": 0.25863406,
        "hits": [
            {
                "_index": ".opensearch-pre-packaged-rules-config",
                "_id": "6KFv1IMBdLpXWBiBelZg",
                "_version": 1,
                "_seq_no": 386,
                "_primary_term": 1,
                "_score": 0.25863406,
                "_source": {
                    "category": "windows",
                    "title": "Change Outlook Security Setting in Registry",
                    "log_source": "registry_set",
                    "description": "Change outlook email security settings",
                    "references": [
                        {
                            "value": "https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1137/T1137.md"
                        },
                        {
                            "value": "https://docs.microsoft.com/en-us/outlook/troubleshoot/security/information-about-email-security-settings"
                        }
                    ],
                    "tags": [
                        {
                            "value": "attack.persistence"
                        },
                        {
                            "value": "attack.t1137"
                        }
                    ],
                    "level": "medium",
                    "false_positives": [
                        {
                            "value": "Administrative scripts"
                        }
                    ],
                    "author": "frack113",
                    "status": "experimental",
                    "last_update_time": "2021-12-28T00:00:00.000Z",
                    "queries": [
                        {
                            "value": "((TargetObject: *\\\\SOFTWARE\\\\Microsoft\\\\Office\\\\*) AND (TargetObject: *\\\\Outlook\\\\Security\\\\*)) AND (EventType: \"SetValue\")"
                        }
                    ],
                    "rule": "title: Change Outlook Security Setting in Registry\nid: c3cefdf4-6703-4e1c-bad8-bf422fc5015a\ndescription: Change outlook email security settings\nauthor: frack113\ndate: 2021/12/28\nmodified: 2022/03/26\nstatus: experimental\nreferences:\n    - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1137/T1137.md\n    - https://docs.microsoft.com/en-us/outlook/troubleshoot/security/information-about-email-security-settings\nlogsource:\n    category: registry_set\n    product: windows\ndetection:\n    selection:\n        TargetObject|contains|all:\n            - '\\SOFTWARE\\Microsoft\\Office\\'\n            - '\\Outlook\\Security\\'\n        EventType: SetValue\n    condition: selection\nfalsepositives:\n    - Administrative scripts\nlevel: medium\ntags:\n  - attack.persistence\n  - attack.t1137\n"
                }
            }
        ]
    }
}
```

---
## Search Custom Rules

### Example request

```json
post/_plugins/_security_analytics/ulues/_search？pre_packaged = false

身体：

{
  "from"：0，
  "size"：20，
  "query"：{
    "nested"：{
      "path"："rule"，
      "query"：{
        "bool"：{
          "must"：[[
            {"match"：{"rule.category"："windows" }}}
          这是给出的
        }
      }
    }
  }
}
```

### Example response

```json
{
    "took"：1，
    "timed_out"： 错误的，
    "_shards"：{
        "total"：1，
        "successful"：1，
        "skipped"：0，
        "failed"：0
    }，，
    "hits"：{
        "total"：{
            "value"：1，
            "relation"："eq"
        }，，
        "max_score"：0.2876821，
        "hits"：[[
            {
                "_index"：".opensearch-custom-rules-config"，
                "_id"："ZaFv1IMBdLpXWBiBa1XI"，
                "_version"：2，
                "_seq_no"：1，
                "_primary_term"：1，
                "_score"：0.2876821，
                "_source"：{
                    "category"："windows"，
                    "title"："Moriya Rooskit"，
                    "log_source"：""，
                    "description"："Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report"，
                    "references"：[[
                        {
                            "value"："https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831"
                        }
                    ]，，
                    "tags"：[[
                        {
                            "value"："attack.persistence"
                        }，，
                        {
                            "value"："attack.privilege_escalation"
                        }，，
                        {
                            "value"："attack.t1543.003"
                        }
                    ]，，
                    "level"："critical"，
                    "false_positives"：[[
                        {
                            "value"："Unknown"
                        }
                    ]，，
                    "author"："Bhabesh Raj"，
                    "status"："experimental"，
                    "last_update_time"："2021-05-06T00:00:00.000Z"，
                    "queries"：[[
                        {
                            "value"："(Provider_Name: \"service_ws_control_ws_manager \") AND (event_uid: 7045) AND (ServiceName: \"zznetsvc \")"
                        }
                    ]，，
                    "rule"："title: Moriya Rooskit\nid: 25b9c01c-350d-4b95-bed1-836d04a4f324\ndescription: Detects the use of Moriya rootkit as described in the securelist's Operation TunnelSnake report\nstatus: experimental\nauthor: Bhabesh Raj\ndate: 2021/05/06\nmodified: 2021/11/30\nreferences:\n    - https://securelist.com/operation-tunnelsnake-and-moriya-rootkit/101831\ntags:\n    - attack.persistence\n    - attack.privilege_escalation\n    - attack.t1543.003\nlogsource:\n    product: windows\n    service: system\ndetection:\n    selection:\n        Provider_Name: 'Service Control Manager'\n        EventID: 7045\n        ServiceName: ZzNetSvc\n    condition: selection\nlevel: critical\nfalsepositives:\n    - Unknown"
                }
            }
        这是给出的
    }
}
```

---
## 删除自定义规则（不强制）

### 示例请求

```json
DELETE /_plugins/_security_analytics/rules/ZaFv1IMBdLpXWBiBa1XI
```

### 示例响应

```json
{
    "error": {
        "root_cause": [
            {
                "type": "security_analytics_exception",
                "reason": "Rule with id ZaFv1IMBdLpXWBiBa1XI is actively used by detectors. Deletion can be forced by setting forced flag to true"
            }
        ],
        "type": "security_analytics_exception",
        "reason": "Rule with id ZaFv1IMBdLpXWBiBa1XI is actively used by detectors. Deletion can be forced by setting forced flag to true",
        "caused_by": {
            "type": "exception",
            "reason": "org.opensearch.OpenSearchStatusException: Rule with id ZaFv1IMBdLpXWBiBa1XI is actively used by detectors. Deletion can be forced by setting forced flag to true"
        }
    },
    "status": 500
}
```

---
## Delete Custom Rule (forced)

### Example request

```json
delete/_plugins/_security_analytics/rules/zafv1imbdlpxwbiba1xi？强制= true = true
```

### Example response

```json
{
    "_id"："ZaFv1IMBdLpXWBiBa1XI"，
    "_version"：1
}
```



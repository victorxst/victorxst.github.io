---
layout: default
title: 审核日志字段参考
parent: 审核日志
nav_order: 130
redirect_from:
  - /security/audit-logs/field-reference/
  - /security-plugin/audit-logs/field-reference/
---

# 审核日志字段参考

此页面包含所有审核日志字段的描述。


## 常见属性

针对所有事件类别记录以下属性，而与该层无关。

姓名| 描述
：--- | ：---
`audit_format_version` | 审核日志消息格式版本。
`audit_category` | 审核日志类别。FAILED_LOGIN，MISSS_PRIVILEGES，BAD_HEADERS，SSL_EXCEPTION，OPENSEARCH_SECURITY_INDEX_ATTEMPT，认证或授予_privileges。
`audit_node_id ` | 生成事件的节点的ID。
`audit_node_name` | 生成事件的节点的名称。
`audit_node_host_address` | 事件生成的节点的主机地址。
`audit_node_host_name` | 生成事件的节点的主机名。
`audit_request_layer` | 发生事件的层，无论是运输还是休息。
`audit_request_origin` | 事件起源的层，即运输或休息。
`audit_request_effective_user_is_admin` | 是的，如果请求是使用TLS管理员证书提出的，否则是错误的。


## REST FAILED_LOGIN属性

姓名| 描述
：--- | ：---
`audit_request_effective_user` | 未能验证的用户名。
`audit_rest_request_path` | 其余端点URI。
`audit_rest_request_params` | HTTP请求参数（如果有）。
`audit_rest_request_headers` | HTTP标头（如果有）。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。


## REST身份验证的属性

姓名| 描述
：--- | ：---
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_rest_request_path` | 其余端点URI。
`audit_rest_request_params` | HTTP请求参数（如果有）。
`audit_rest_request_headers` | HTTP标头（如果有）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。


## REST SSL_EXCEPTION属性

姓名| 描述
：--- | ：---
`audit_request_exception_stacktrace` | SSL异常的堆栈轨迹。


## REST BAD_HEADERS属性

姓名| 描述
：--- | ：---
`audit_rest_request_path` | 其余端点URI。
`audit_rest_request_params` | HTTP请求参数（如果有）。
`audit_rest_request_headers` | HTTP标头（如果有）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。


## 传输失败_login属性

姓名| 描述
：--- | ：---
`audit_trace_task_id` | 请求的ID。
`audit_transport_headers` | 请求的标题（如果有）。
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_transport_request_type` | 请求的类型（例如`IndexRequest`）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。
`audit_trace_indices` | 请求中包含索引名称。可以包含通配符，日期模式和别名。仅记录`resolve_indices` 是真的。
`audit_trace_resolved_indices` | 已解决的索引名称受请求影响。仅记录`resolve_indices` 是真的。
`audit_trace_doc_types` | 受请求影响的文档类型。仅记录`resolve_indices` 是真的。


## 运输验证的属性

姓名| 描述
：--- | ：---
`audit_trace_task_id` | 请求的ID。
`audit_transport_headers` | 请求的标题（如果有）。
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_transport_request_type` | 请求的类型（例如`IndexRequest`）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。
`audit_trace_indices` | 请求中包含索引名称。可以包含通配符，日期模式和别名。仅记录`resolve_indices` 是真的。
`audit_trace_resolved_indices` | 已解决的索引名称受请求影响。仅记录`resolve_indices` 是真的。
`audit_trace_doc_types` | 受请求影响的文档类型。仅记录`resolve_indices` 是真的。


## 传输丢失_privileges属性

姓名| 描述
：--- | ：---
`audit_trace_task_id` | 请求的ID。
`audit_trace_task_parent_id` | 此请求的父ID（如果有）。
`audit_transport_headers` | 请求的标题（如果有）。
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_transport_request_type` | 请求的类型（例如`IndexRequest`）。
`audit_request_privilege` | 请求的要求特权（例如，`indices:data/read/search`）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。
`audit_trace_indices` | 请求中包含索引名称。可以包含通配符，日期模式和别名。仅记录`resolve_indices` 是真的。
`audit_trace_resolved_indices` | 已解决的索引名称受请求影响。仅记录`resolve_indices` 是真的。
`audit_trace_doc_types` | 受请求影响的文档类型。仅记录`resolve_indices` 是真的。


## 运输授予的_privileges属性

姓名| 描述
：--- | ：---
`audit_trace_task_id` | 请求的ID。
`audit_trace_task_parent_id` | 此请求的父ID（如果有）。
`audit_transport_headers` | 请求的标题（如果有）。
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_transport_request_type` | 请求的类型（例如，`IndexRequest`）。
`audit_request_privilege` | 请求的要求特权（例如`indices:data/read/search`）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。
`audit_trace_indices` | 请求中包含索引名称。可以包含通配符，日期模式和别名。仅记录`resolve_indices` 是真的。
`audit_trace_resolved_indices` | 已解决的索引名称受请求影响。仅记录`resolve_indices` 是真的。
`audit_trace_doc_types` | 受请求影响的文档类型。仅记录`resolve_indices` 是真的。


## 传输ssl_exception属性

姓名| 描述
：--- | ：---
`audit_request_exception_stacktrace` | SSL异常的堆栈轨迹。


## 传输bad_headers属性

姓名| 描述
：--- | ：---
`audit_trace_task_id` | 请求的ID。
`audit_trace_task_parent_id` | 此请求的父ID（如果有）。
`audit_transport_headers` | 请求的标题（如果有）。
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_transport_request_type` | 请求的类型（例如`IndexRequest`）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。
`audit_trace_indices` | 请求中包含索引名称。可以包含通配符，日期模式和别名。仅记录`resolve_indices` 是真的。
`audit_trace_resolved_indices` | 已解决的索引名称受请求影响。仅记录`resolve_indices` 是真的。
`audit_trace_doc_types` | 受请求影响的文档类型。仅记录`resolve_indices` 是真的。


## 运输opensearch_security_index_attempt属性

姓名| 描述
：--- | ：---
`audit_trace_task_id` | 请求的ID。
`audit_transport_headers` | 请求的标题（如果有）。
`audit_request_effective_user` | 未能验证的用户名。
`audit_request_initiating_user` | 启动请求的用户。仅当它与有效用户不同时才记录。
`audit_transport_request_type` | 请求的类型（例如`IndexRequest`）。
`audit_request_body` | HTTP请求主体，如果有任何（以及启用了请求的车身记录）。
`audit_trace_indices` | 请求中包含索引名称。可以包含通配符，日期模式和别名。仅记录`resolve_indices` 是真的。
`audit_trace_resolved_indices` | 已解决的索引名称受请求影响。仅记录`resolve_indices` 是真的。
`audit_trace_doc_types` | 受请求影响的文档类型。仅记录`resolve_indices` 是真的。


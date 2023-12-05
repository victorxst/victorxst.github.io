---
layout: default
title: key_value
parent: 处理器
grand_parent: 管道
nav_order: 56
---

# key_value


您可以使用`key_value` 处理器将指定字段解析为密钥-价值对。您可以自定义`key_value` 使用以下选项解析现场信息的处理器。以下每个选项的类型是`string`。

| 选项| 描述| 例子|
| ：--- | ：--- | ：--- |
| 来源| 要解析的消息字段。选修的。默认值是`message`。| 如果`source` 是`"message1"`，，，，`{"message1": {"key1=value1"}, "message2": {"key2=value2"}}` 解析`{"message1": {"key1=value1"}, "message2": {"key2=value2"}, "parsed_message": {"key1": "value1"}}`。|
| 目的地| 解析源的目标字段。解析的源覆盖该密钥的预先存在数据。选修的。如果`destination` 被设定为`null`，解析的字段将写入事件的根源。默认值是`parsed_message`。| 如果`destination` 是`"parsed_data"`，，，，`{"message": {"key1=value1"}}` 解析`{"message": {"key1=value1"}, "parsed_data": {"key1": "value1"}}`。|
| field_delimiter_regex| 指定分开键的定界符的正则表达式-价值对。特殊的正则表达字符，例如`[` 和`]` 必须逃脱`\\`。不能与`field_split_characters`。选修的。如果未定义此选项，`field_split_characters` 用来。| 如果`field_delimiter_regex` 是`"&\\{2\\}"`，，，，`{"key1=value1&&key2=value2"}` 解析`{"key1": "value1", "key2": "value2"}`。|
| field_split_characters| 一串字符，指定分开密钥的分隔线-价值对。特殊的正则表达字符，例如`[` 和`]` 必须逃脱`\\`。不能与`field_delimiter_regex`。选修的。默认值是`&`。| 如果`field_split_characters` 是`"&&"`，，，，`{"key1=value1&&key2=value2"}` 解析`{"key1": "value1", "key2": "value2"}`。|
| key_value_delimiter_regex| 指定分隔符的正则表达式，该定界符将密钥和值分开-价值对。特殊的正则表达字符，例如`[` 和`]` 必须逃脱`\\`。此选项不能与`value_split_characters`。选修的。如果未定义此选项，`value_split_characters` 用来。| 如果`key_value_delimiter_regex` 是`"=\\{2\\}"`，，，，`{"key1==value1"}` 解析`{"key1": "value1"}`。|
| value_split_characters| 指定定界符的字符串，该字符将密钥和值分开的定界符-价值对。特殊的正则表达字符，例如`[` 和`]` 必须逃脱`\\`。不能与`key_value_delimiter_regex`。选修的。默认值是`=`。| 如果`value_split_characters` 是`"=="`，，，，`{"key1==value1"}` 解析`{"key1": "value1"}`。|
| non_match_value| 当钥匙时-价值对不能成功拆分，键-价值对放在`key` 字段，指定的值放在`value` 场地。选修的。默认值是`null`。| `key1value1&key2=value2` 解析`{"key1value1": null, "key2": "value2"}`。|
| 字首| 在所有密钥之前附加附加的前缀。选修的。默认值是一个空字符串。| 如果`prefix` 是`"custom"`，，，，`{"key1=value1"}` 解析`{"customkey1": "value1"}`。|
| delete_key_regex| 正则表达式指定字符从密钥中删除。特殊的正则表达字符，例如`[` 和`]` 必须逃脱`\\`。不能是一个空字符串。选修的。没有默认值。| 如果`delete_key_regex` 是`"\s"`，，，，`{"key1 =value1"}` 解析`{"key1": "value1"}`。|
| delete_value_regex| 正则表达式指定字符从值中删除。特殊的正则表达字符，例如`[` 和`]` 必须逃脱`\\`。不能是一个空字符串。选修的。没有默认值。| 如果`delete_value_regex` 是`"\s"`，，，，`{"key1=value1 "}` 解析`{"key1": "value1"}`。|
| include_keys| 一个指定应添加的键的数组。默认情况下，将添加所有密钥。| 如果`include_keys` 是`["key2"]`，，，，`key1=value1&key2=value2` 会解析`{"key2": "value2"}`。|
| dubl_keys| 指定不应添加到事件的分析键的数组。默认情况下，不会排除任何密钥。| 如果`exclude_keys` 是`["key2"]`，，，，`key1=value1&key2=value2` 会解析`{"key1": "value1"}`。|
| 默认值| 指定默认键及其值应添加到事件中的地图，以防这些键在解析的源字段中不存在。如果消息中已经存在默认密钥，则该值不会更改。这`include_keys` 过滤器将在消息之前应用于消息`default_values`。| 如果`default_values` 是`{"defaultkey": "defaultvalue"}`，，，，`key1=value1` 会解析`{"key1": "value1", "defaultkey": "defaultvalue"}`。<br /> if`default_values` 是`{"key1": "abc"}`，，，，`key1=value1` 会解析`{"key1": "value1"}`。<br /> if`include_keys` 是`["key1"]` 和`default_values` 是`{"key2": "value2"}`，，，，`key1=value1&key2=abc` 会解析`{"key1": "value1", "key2": "value2"}`。|
| transform_key| 何时低写，大写或大写密钥。| 如果`transform_key` 是`lowercase`，，，，`{"Key1=value1"}` 会解析`{"key1": "value1"}`。<br /> if`transform_key` 是`uppercase`，，，，`{"key1=value1"}` 会解析`{"KEY1": "value1"}`。<br /> if`transform_key` 是`capitalize`，，，，`{"key1=value1"}` 会解析`{"Key1": "value1"}`。|
| 空格| 指定是否要宽大还是严格，以接受已配置的值的不必要的空白空间-拆分序列。默认为`lenient`。| 如果`whitespace` 是`"lenient"`，，，，`{"key1  =  value1"}` 会解析`{"key1  ": "  value1"}`。如果`whitespace` 是`"strict"`，，，，`{"key1  =  value1"}` 会解析`{"key1": "value1"}`。|
| skip_duplate_values| 删除重复密钥的布尔值选项-价值对。设置为`true`，只有一个唯一的钥匙-价值对将保留。默认为`false`。| 如果`skip_duplicate_values` 是`false`，，，，`{"key1=value1&key1=value1"}` 会解析`{"key1": ["value1", "value1"]}`。如果`skip_duplicate_values` 是`true`，，，，`{"key1=value1&key1=value1"}` 会解析`{"key1": "value1"}`。|
| remove_brackets| 指定是否将方括号，角括号和括号视为值"wrappers" 应从值中删除。默认为`false`。| 如果`remove_brackets` 是`true`，，，，`{"key1=(value1)"}` 会解析`{"key1": value1}`。如果`remove_brackets` 是`false`，，，，`{"key1=(value1)"}` 会解析`{"key1": "(value1)"}`。|
| 递归| 指定是否要递归获得其他密钥-值对值。额外的钥匙-价值对将存储为子-根键的键。默认为`false`。递归解析的水平必须由每个级别的不同括号定义：`[]`，，，，`()`， 和`<>`，按此顺序。指定的任何其他配置都只能应用于最大键。<br />何时`recursive` 是`true`：<br />`remove_brackets` 也不是`true`; <br />`skip_duplicate_values` 一直会`true`;<br />`whitespace` 一直会`"strict"`。| 如果`recursive` 是真的，`{"item1=[item1-subitem1=item1-subitem1-value&item1-subitem2=(item1-subitem2-subitem2A=item1-subitem2-subitem2A-value&item1-subitem2-subitem2B=item1-subitem2-subitem2B-value)]&item2=item2-value"}` 会解析`{"item1": {"item1-subitem1": "item1-subitem1-value", "item1-subitem2" {"item1-subitem2-subitem2A": "item1-subitem2-subitem2A-value", "item1-subitem2-subitem2B": "item1-subitem2-subitem2B-value"}}}`。|
| operwrite_if_destination_exists| 指定是否在为事件编写解析字段时是否存在关键冲突，是否覆盖现有字段。默认为`true`。| 如果`overwrite_if_destination_exists` 是`true` 目的地是`null`，，，，`{"key1": "old_value", "message": "key1=new_value"}` 会解析`{"key1": "new_value", "message": "key1=new_value"}`。|
| tags_on_failure| 当一个`kv` 操作会导致处理器内的运行时异常，该操作将安全停止而不会崩溃处理器，并且该事件用提供的标签标记。| 如果`tags_on_failure` 被设定为`["keyvalueprocessor_failure"]`，，，，`{"tags": ["keyvalueprocessor_failure"]}` 如果运行时例外，将添加到活动的元数据中。|



<！--- ## 配置

内容将添加到本节中。

## 指标

内容将添加到本节中。---


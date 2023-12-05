---
layout: default
title: 表达语法
parent: 管道
nav_order: 12
---

# 表达语法

以下各节提供了有关Data Prepper中表达语法的信息。

## 支持运营商

按照优先顺序列出操作员（上到底，从左到右）。

| 操作员| 描述| 关联|
|----------------------|-------------------------------------------------------|---------------|
| `()`                 | 优先表达| 左边-到-正确的|
| `not`<br>`+`<br>`-`| 一级逻辑不是<br>一元阳性<br>单位负面| 正确的-到-左边|
| `<`，`<=`，`>`，`>=` | 关系运营商| 左边-到-正确的|
| `==`，`!=`           | 平等操作员| 左边-到-正确的|
| `and`，`or`          | 条件表达| 左边-到-正确的|

## 保留用于可能的未来功能

保留符号集：`^`，`*`，`/`，`%`，`+`，`-`，`xor`，`=`，`+=`，`-=`，`*=`，`/=`，`%=`，`++`，`--`，`${<text>}`

## 设置初始化器

集合初始化器定义集合或术语和/或表达式。

### 例子

以下是设置初始化器语法的示例。

#### HTTP状态代码

```
{200, 201, 202}
```

#### HTTP响应有效载荷

```
{"Created", "Accepted"}
```

#### 用不同的键处理多种事件类型

```
{/request_payload, /request_message}
```

## 优先表达

优先表达式标识将在最高优先级评估的表达式。优先表达式必须包含表达式或值；不支持空括号。

### 例子

```
/is_cool == (/name == "Steven")
```

## 关系运营商

关系运算符用于测试两个数字值的关系。操作数必须是溶于数字的数字或JSON指针。

### 句法

```
<Number | JSON Pointer> < <Number | JSON Pointer>
<Number | JSON Pointer> <= <Number | JSON Pointer>
<Number | JSON Pointer> > <Number | JSON Pointer>
<Number | JSON Pointer> >= <Number | JSON Pointer>
```

### 例子

```
/status_code >= 200 and /status_code < 300
```

## 平等操作员

平等运算符用于测试两个值是否等效。

### 句法
```
<Any> == <Any>
<Any> != <Any>
```

### 例子
```
/is_cool == true
3.14 != /status_code
{1, 2} == /event/set_property
```
## 使用平等操作员检查JSON指针

平等运算符也可以用于检查JSON指针是否通过将值与`null`。

### 句法
```
<JSON Pointer> == null
<JSON Pointer> != null
null == <JSON Pointer>
null != <JSON Pointer>
```

### 例子
```
/response == null
null != /response
```

#### 条件表达

有条件的表达式用于将多个表达式和/或值链在一起。

#### 句法
```
<Any> and <Any>
<Any> or <Any>
not <Any>
```

### 例子
```
/status_code == 200 and /message == "Hello world"
/status_code == 200 or /status_code == 202
not /status_code in {200, 202}
/response == null
/response != null
```

## 定义

本节提供表达定义。

### 文字
字面意思是没有孩子的基本价值：
- 浮点：支持3.40282347＆Times的值；10 <sup> 38 </sup>至1.40239846＆times;10 <sup>＆minus; 45 </sup>。
- 整数：支持＆Minus的值; 2,147,483,648至2,147,483,647。
- 布尔值：支持真或错误。
- JSON指针：请参阅[JSON指针](#json-pointer) 详细信息部分。
- 字符串：支持有效的Java字符串。
- NULL：支持Null检查是否存在JSON指针。

### 表达式字符串
表达式字符串在数据PEPPER表达式中获得最高优先级，并且仅支持一个表达式字符串，从而导致返回值。_expression String_与_expression_不同。

### 陈述
陈述是最高的-表达字符串的优先组件。

### 表达
表达式是包含_primary_或_operator_的通用组件。表达可能包含表达式。表情的迫在眉睫的孩子可以包含0-1个_operators_。

### 基本的

- _放_
- _ priority expression _
- _文字_

### 操作员
操作员是一个硬编码令牌，可以标识_expression_中使用的操作。

### JSON指针
JSON指针是一种文字，用于在事件中引用一个值，并作为_expression String_的上下文提供。JSON指针由领先者确定`/` 包含字母数字字符或下划线，由`/`。如果用双引号包装，JSON Pointers可以使用扩展字符集`"`）使用逃生角色`\`。请注意，JSON指针需要`~` 和`/` 字符，应用作路径的一部分，而不是需要逃脱的定界符。

以下是JSON指针的例子：

- `~0` 代表`~`
- `~1` 代表`/`

#### 速记语法（正则`\w` =`[A-Za-z_]`）
```
/\w+(/\w+)*
```

#### 速记的例子

以下是速记的一个例子：

```
/Hello/World/0
```

#### 逃脱语法的示例

以下是逃脱语法的示例：
```
"/<Valid String Characters | Escaped Character>(/<Valid String Characters | Escaped Character>)*"
```

#### 逃脱的JSON指针的示例

以下是逃脱的JSON指针的示例：
```
# Path
# { "Hello - 'world/" : [{ "\"JsonPointer\"": true }] }
"/Hello - 'world\//0/\"JsonPointer\""
```

## 空白

空白是**选修的** 周围的关系运营商，正则等级运营商，平等操作员和逗号。
空白是**必需的** 周围设置的初始化器，优先表达式，集合运算符和条件表达式。


| 操作员| 描述| 需要空白| ✅有效示例| ❌无效的例子|
|----------------------|--------------------------|----------------------|----------------------------------------------------------------|---------------------------------------|
| `{}`                 | 设置初始化器| 是的| `/status in {200}`                                             | `/status in{200}`                     |
| `()`                 | 优先表达| 是的| `/a==(/b==200)`<br>`/a in ({200})`                             | `/status in({200})`                   |
| `in`，`not in`       | 设置运算符| 是的| `/a in {200}`<br>`/a not in {400}`                             | `/a in{200, 202}`<br>`/a not in{400}` |
| `<`，`<=`，`>`，`>=` | 关系运营商| 不| `/status < 300`<br>`/status>=300`                              |                                       |
| `=~`，`!~`           | 正则平等pperators| 不| `/msg =~ "^\w*$"`<br>`/msg=~"^\w*$"`                           |                                       |
| `==`，`!=`           | 平等操作员| 不| `/status == 200`<br>`/status_code==200`                        |                                       |
| `and`，`or`，`not`   | 有条件的操作员| 是的| `/a<300 and /b>200`                                            | `/b<300and/b>200`                     |
| `,`                  | 设置值定界符| 不| `/a in {200, 202}`<br>`/a in {200,202}`<br>`/a in {200 , 202}` | `/a in {200,}`                        |


## 功能

数据预先支持以下构建-在表达式中可以使用的函数中。

### `length()`

这`length()` 函数采用JSON指针类型的一个参数，并返回传递的值的长度。例如，`length(/message)` 返回长度`10` 当事件中存在关键信息并具有`1234567890`。

### `hasTags()`

这`hastags()` 功能采用一个或多个字符串类型参数并返回`true` 如果所有通过的参数都存在于事件标签中。当事件标签中不存在参数时，函数返回`false`。例如，如果您使用表达式`hasTags("tag1")` 活动包含`tag1`，数据预先回报`true`。如果您使用表达式`hasTags("tag2")` 但是该事件只包含一个`tag1` 标签，数据预先返回`false`。

### `getMetadata()`

这`getMetadata()` 功能需要一个字面的字符串参数，以查找事件的元数据中的特定键。如果键包含一个`/`，然后该函数递归地抬起元数据。通过时，表达式返回对应于键的值。返回的值可以是任何类型。例如，如果元数据包含`{"key1": "value2", "key2": 10}`，然后函数，`getMetadata("key1")`，返回`value2`。功能，`getMetadata("key2")`，返回10。

### `contains()`

这`contains()` 函数采用两个字符串参数，并确定事件中是否包含文字字符串或JSON指针。当第二个参数包含第一个参数的子字符串时，例如`contains("abcde", "abcd")`，函数返回`true`。如果第二个参数不包含任何子字符串，例如`contains("abcde", "xyz")`，它返回`false`。

### `cidrContains()`

这`cidrContains()` 功能需要两个或多个参数。第一个参数是JSON指针，它代表了已检查的IP地址的键。它支持IPv4和IPv6地址。密钥之后发生的每个参数都是一个字符串类型，代表被检查的CIDR块。

如果第一个参数中的IP地址在给定CIDR块的任何一个范围内，则该函数返回`true`。如果IP地址不在CIDR块的范围内，则该功能返回`false`。例如，`cidrContains(/sourceIp,"192.0.2.0/24","10.0.1.0/16")` 将返回`true` 如果是`sourceIp` JSON指针中指示的字段具有`192.0.2.5`。


---
layout: default
title: 混淆
parent: 处理器
grand_parent: 管道
nav_order: 71
---

# 混淆

这`obfuscate` 流程使文档中的字段混淆以保护敏感数据。

## 用法

在此示例中，文档包含一个`log` 字段和a`phone` 字段，如以下对象所示：

```json
{
  "id": 1,
  "phone": "(555) 555 5555",
  "log": "My name is Bob and my email address is abc@example.com"
}
```


混淆`log` 和`phone` 字段，添加`obfuscate` 处理器并调用每个字段`source` 选项。考虑到这两个`log` 和`phone` 字段，以下示例使用多个`obfuscate` 处理器因为每个处理器只能混淆一个来源。

在第一个`obfuscate` 管道中的处理器，来源`log` 如下所示，使用多个配置选项掩盖了日志字段中的数据。有关这些选项的更多详细信息，请参阅[配置](#configuration)。

```yaml
pipeline:
  source:
    http:
  processor:
    - obfuscate:
        source: "log"
        target: "new_log"
        patterns:
          - "[A-Za-z0-9+_.-]+@([\\w-]+\\.)+[\\w-]{2,4}"
        action:
          mask:
            mask_character: "#"
            mask_character_length: 6
    - obfuscate:
        source: "phone"
  sink:
    - stdout:
```

运行时，`obfuscate` 处理器将字段解析为以下输出：

```json
{
  "id": 1,
  "phone": "***",
  "log": "My name is Bob and my email address is abc@example.com",
  "newLog": "My name is Bob and my email address is ######"
}
```

## 配置

使用以下配置选项与`obfuscate` 处理器。

| 范围| 必需的| 描述|
| :--- | :---  | ：---  |
| `source` | 是的| 混淆的源字段。|
| `target` | 不| 存储混淆值的新字段。这使原始源场不变。当没有`target` 提供了带有混淆值的源字段更新。|
| `patterns` | 不| 允许您混淆字段特定部分的正则方式列表。只有匹配正则图案的零件才会混淆。如果没有提供，则处理器使整个领域混淆。|
| `action` | 不| 混淆动作。从数据Prepper 2.3开始，只有`mask` 支持行动。|

您可以自定义`mask` 采取以下可选配置选项的操作。

| 范围| 默认| 描述|
| :--- | :---  | ：---  |
`mask_character` | `*` | 掩盖时使用的字符。有效字符是！，#，$，％，＆， *和 @。|
`mask_character_length` | `3` | 在现场掩盖字符的数量。值必须在1到10之间。|


## 预定义的模式

使用时`patterns` 配置选项，您可以为公共字段使用一组预定义的混淆模式。这`obfuscate` 处理器支持以下预定义的模式。

您不能为一个混淆处理器使用多种模式。为每个混淆处理器使用一种模式。
{: .note}


| 图案名称| 例子|
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ％{电子邮件地址}| abc@test.com <br/> 123@test.com <br/> abc123@test.com <br/> abc_123@test.com <br/> a-b@test.com <br/> a.b@test.com <br/> abc@test-test.com <br/> abc@test.com.cn <br/> abc@test.mail.com.org|
| ％{ip_address_v4}| 1.1.1.1 <br/> 192.168.1.1 <br/> 255.255.255.0|
| ％{base_number}| 1.1 <br/> .1 <br/> 2000|
| ％{信用卡号码}| 5555555555554444 <br/> 41111111111111111 <br/> 1234567890123456 <br/> 1234 5678 9012 3456 <br/> 1234-5678-9012-3456|
| ％{us_phone_number}| 1555 5555555 <br/> 5555555555 <br/> 1-555-555-5555 <br/> 1-（555）-555-5555 <br/> 1（555）555 5555 <br/>（555）555 5555 <br/> +1-555-555-5555 <br/>|
| ％{US_SSN_NUMBER}| 123-11-1234


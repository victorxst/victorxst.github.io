---
layout: default
title: 处理检测规则
parent: 使用安全分析
nav_order: 40
---

# 处理检测规则

这**检测规则** 窗口列出了用于检测创建的所有安全规则，并提供了用于过滤列表和查看每个规则的详细信息的选项。更多的选项使您可以通过首先复制Sigma规则，然后对其进行修改，从而导入规则并创建新规则。本节涵盖了**规则** 页面并提供您可以执行的操作的描述。

<img src="{{site.url}}{{site.baseurl}}/images/Security/Rules.png" alt="The Rules page" width="90%">

---
## Viewing and filtering rules

当你打开**检测规则** 页面，所有规则均在表中列出。使用搜索栏通过输入完整或部分名称并按下来搜索特定规则**返回/输入** 在键盘上。列表已过滤并显示匹配结果。

或者，您可以使用**规则类型**，**规则严重性**， 和**来源** 下拉列表要在警报中钻取并过滤以获得首选结果。您可以从每个列表中选择多个选项，并将所有三个组合使用以缩小结果。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/rule-menu.png" alt ="Rule menus for filtering results" width="40%">

### Rule details

要查看规则详细信息，请在列表的规则名称列中选择规则。规则详细信息窗格开放。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/Rule_details.png" alt ="The rule details pane" width="50%">

在视觉视图中，规则详细信息在字段中排列，链接是活动的。选择**yaml** 以yaml文件格式显示规则。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/rule_detail_yaml.png" alt ="The rule details pane in YAML file view" width="50%">

*规则详细信息根据Sigma规则规范格式化为YAML文件。
*要复制规则，请在上部选择复制图标-规则的右角。要快速创建一个新的自定义规则，您可以将规则粘贴到YAML编辑器中，并在保存之前进行任何修改。请参阅[自定义规则]（#定制-规则）有关更多信息。

---
## 创建检测规则

有多种方法可以在**检测规则** 页。这些方法包括手动创建自定义规则，导入规则，并重复现有规则以自定义它。以下各节详细讨论了这些方法。

### 自定义规则

规则创建的第一种方法是通过使用Visual Editor或YAML编辑器手动填写完成规则的必要字段来创建自定义规则。为此，选择**创建检测规则** 在超你-屏幕的右角。这**创建检测规则** 窗口打开。

如果您选择手动创建规则，则可以参考Sigma的[规则创建指南](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide) 有关每个字段的详细信息的更多信息。
{: .tip }

#### 视觉编辑器

当。。。的时候**创建检测规则** 窗口打开，**视觉编辑器** 默认情况下显示。所需的字段**视觉编辑器** 对应于格式为Sigma规则的YAML文件中找到的基本字段。这些步骤中的描述提到了此对应关系，当它可能立即明显时。
  
1. 在里面**规则概述** 部分，输入规则的名称，描述（可选）和规则的作者。这**规则名称** 对应于[标题](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide#title) 在YAML文件中格式化的Sigma规则中。下图提供了填充字段的示例。
  
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/overview-rule.png" alt ="The Rule overview fields in the Create detection rule window, which include the rule name, description, and author fields." width="50%">
  
1. 在里面**细节** 部分，输入数据源，规则级别和规则状态的日志类型。这**日志类型** 对应于[`logsource`](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide#log-source) 字段（特别是`logsource: product` 字段），而规则级别和规则状态对应于[`level`](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide#level) 和[`status`](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide#status)， 分别。Sigma规则中的级别包括 *信息 *， *低 *， *中 *， *高 *和 *关键 *。下图提供了一个示例。
  
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/details-rule.png" alt ="The Details fields in the Create detection rule window, which include the log type, rule level, and rule status fields." width="40%">
  
1. 在里面**检测** 部分，指定密钥-值对表示对数源中的字段及其值，这将是检测的目标。这些钥匙-值对定义检测。您可以将钥匙值表示为单个值或包含多个值的列表。
   
   定义一个简单的密钥-价值对，首先将光标放在**选择_1** 标记并用描述密钥的选择名称替换-价值对。接下来，从日志源输入首选字段**钥匙**，然后使用**修饰符** 下拉列表以定义如何处理该值。可用以下修饰符：
     *`contains`  - 在值的两侧添加通配符，以便将其匹配到现场的任何位置。
     *`all`  - 在列表的情况下，而不是使用或逻辑单独的值，逻辑就会成为并寻找与所有值的匹配。
     *`endswith`  - 指示该值出现在字段末端时匹配。
     *`startswith`  - 指示该值出现在字段开头时匹配。
   
   选择修饰符后，选择**价值** 广播按钮，然后在随后的文本字段中输入键的值。
   
   您可以添加字段以映射第二个键-通过选择价值对**添加地图**。在此步骤中遵循以前的指南以映射密钥-价值对。下图显示了两个键的定义如何-价值对出现在**创建检测规则** 窗户。
   
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/detection1.png" alt ="An example of the Detection fields." width="50%">
   
   要查看此定义与YAML文件中如何配置的定义相比，请参阅以下示例：

   ```yaml
   detection:
      selection:
       selection_schtasks:
         Image|endswith: \schtasks.exe
         CommandLine|contains: '/Create '
   ```
   
   要添加第二个选择，请使用**添加选择** 第一个选择之后的栏以打开另一个钥匙-价值对映射。对于此选择，将值作为列表提供。如第一个选择中所述，替换**选择_2** 标签带有选择名称的标签，从日志中输入字段名称作为密钥，然后从**修饰符** 下拉列表。

   然后，定义钥匙-使用列表而不是单个值的值对，选择**列表** 单选按钮。这**上传文件** 出现按钮并扩展文本框以适应列表。

   您可以以.csv或.txt格式上传现有值列表。选择**上传文件** 并按照提示将文件的内容上传到文本字段中。作为替代方案，您可以直接在文本字段中手动撰写列表。下图显示了如何键-出现包括值列表的值对映射。

   <img src ="{{site.url}}{{site.baseurl}}/images/Security/detection2.png" alt ="An example of the Detection fields." width="50%">
   
   要查看使用两个选择的定义与YAML文件中如何配置的定义相比，请参阅以下示例：

   ```yml
   detection:
     selection:
       selection_schtasks:
         Image|endswith: \schtasks.exe
         CommandLine|contains: '/Create '
       selection_rare:
         CommandLine|contains:
         - ' bypass '
         - .DownloadString
         - .DownloadFile
         - FromBase64String
         - ' -w hidden '
         - ' IEX'
         - ' -enc '
         - ' -decode '
         - '/c start /min '
         - ' curl '
   ```

1. In the **Condition** section, specify the conditions for the selections included in the detection definition. These conditions determine how the defined selections are handled by the detection rule. At least one selection is required. In the case of the preceding example, this means that at least one of the two selections `selection_schtasks` and `selection_rare` 必须在**状况** 部分。

   选择`+` 在旁边签名**选择** 添加第一个选择。选择`+` 再次签名以从检测定义中添加进一步的选择。一旦有两个选择作为条件，布尔运算符并在它们之间出现，表明两者都将在检测规则查询中使用。您可以选择操作员的标签以打开操作员下拉列表并从选项中选择`AND`，`OR`， 和`NOT`。下图显示了此选项的出现。

   <img src ="{{site.url}}{{site.baseurl}}/images/Security/condition1.png" alt ="specifying the conditions for the selections in the detection definition." width="50%">

1. 为检测规则指定可选字段。
  
   * 在里面**标签** 部分，添加标签以将检测规则与网络安全知识基础记录的任何攻击技术相关联[MITER ATT＆CK](https://attack.mitre.org/)。选择**添加标签** 添加多个标签。
   * 在里面**参考** 部分，您可以为规则参考添加URL。选择**添加URL** 添加多个URL。
   * 这**假阳性案件** 部分提供了列出假积极条件的描述的空间，这些条件可能会触发该规则不必要的警报。选择**添加假阳性** 添加多个描述。
  
 1.一旦规则完成并满足您的要求，请选择**创建检测规则** 在较低-窗口的右角保存规则。规则ID自动分配给新规则，并显示在检测规则列表中。
  
#### YAML编辑

这**创建检测规则** 窗口还包含YAML编辑器，因此您可以直接以YAML文件格式创建一个新规则。选择**YAML编辑** 然后输入PRE的信息-填充的字段类型。规则`id` 保存规则时提供并分配。以下示例显示了典型规则的基本元素：

```yml
title: RDP Sensitive Settings Changed
logsource:
  product: windows
description: 'Detects changes to RDP terminal service sensitive settings'
detection:
  selection:
    EventType: SetValue
    TargetObject|contains:
      - \services\TermService\Parameters\ServiceDll
      - \Control\Terminal Server\fSingleSessionPerUser
      - \Control\Terminal Server\fDenyTSConnections
      - \Policies\Microsoft\Windows NT\Terminal Services\Shadow
      - \Control\Terminal Server\WinStations\RDP-Tcp\InitialProgram
  condition: selection
level: high
tags:
  - attack.defense_evasion
  - attack.t1112
references:
  - https://blog.menasec.net/2019/02/threat-hunting-rdp-hijacking-via.html
  - https://knowledge.insourcess.com/Supporting_Technologies/Wonderware/Tech_Notes/TN_WW213_How_to_shadow_an_established_RDP_Session_on_Windows_10_Pro
  - https://twitter.com/SagieSec/status/1469001618863624194?t=HRf0eA0W1YYzkTSHb-Ky1A&s=03
  - http://etutorials.org/Microsoft+Products/microsoft+windows+server+2003+terminal+services/Chapter+6+Registry/Registry+Keys+for+Terminal+Services/
falsepositives:
  - Unknown
author:
  - Samir Bousseaden 
  - David ANDRE
status: experimental
```
{% include copy.html %}

使用**YAML编辑**，您可以参考Sigma的[规则创建指南](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide) 并使用每个字段的描述以了解有关定义规则的更多信息。

### 导入规则

安全分析还支持以YAML格式进口Sigma规则。在里面**检测规则** 窗口，请按照以下步骤导入规则。

1. 首先，选择**进口检测规则** 在上部-页面的右角。这**导入规则** 页面打开。
1. 要么拖山药-通过选择链接并打开链接，将Sigma规则格式化为窗口或浏览文件。这**导入规则** 打开窗口，规则定义字段在Visual Editor和YAML编辑器中自动填充。
1. 验证或修改字段中的信息。
1. 确认规则的信息准确后，请选择**创建检测规则** 在较低-窗户的右角。创建了一个新规则，并显示在检测规则列表中。

### 定制规则

创建新检测规则的另一个选项是复制Sigma规则，然后对其进行修改以创建自定义规则。首先搜索或过滤规则**规则名称** 列表以找到要复制的规则。下图显示了用关键字过滤的列表。

<img src="{{site.url}}{{site.baseurl}}/images/Security/rules-dup1.png" alt="Selecting a rule in the Rules name list" width="75%">

1. 首先，选择规则**规则名称** 柱子。显示规则详细信息。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/rule-dup2.png" alt ="Opening the rule details pane" width="50%">

1. 选择**复制** upper-窗格的右角。“重复规则”窗口在Visual Editor视图中打开，所有字段都会自动填充规则的详细信息。YAML编辑器视图中还填充了详细信息。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/dupe-rule.png" alt ="Selecting the duplicate button opens the Duplicate rule window" width="50%">

1. 在Visual Editor视图或YAML编辑器视图中，修改任何字段以自定义规则。
1. 对规则执行任何修改后，选择**创建检测规则** 在较低-窗户的右角。创建了一个新的自定义规则。它显示在规则列表中的主页上**检测规则** 窗户。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/custom-rule.png" alt ="The custom rule now appears in the list of rules." width="70%">

您无法修改Sigma规则本身。原始的Sigma规则始终保留在系统中。在修改后，其重复将其重复成为添加到规则列表中的自定义规则。
{: .note }



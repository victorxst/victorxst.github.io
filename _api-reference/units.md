---
layout: default
title: 支持单位
nav_order: 90
redirect_from:
  - /opensearch/units/
---

# 支持单位
**引入1.0**
{: .label .label-purple }

OpenSearch支持所有REST操作的以下单元：

单元| 描述| 例子
:--- | :--- | :---
时代| 随时间的支持单位是`d` 好几天，`h` 用了几个小时，`m` 几分钟`s` 几秒钟`ms` 对于毫秒`micros` 对于微秒，以及`nanos` 对于纳秒。| `5d` 或者`7h`
字节| 字节大小的支持单元是`b` 对于字节，`kb` 对于kibibytes，`mb` 对于Mebibytes，`gb` 对于gibibytes，`tb` 对于tebibytes和`pb` 用于卵石。尽管有基地-10个缩写，这些单位是基础-2;`1kb` 是1,024个字节，`1mb` 是1,048,576个字节，等。| `7kb` 或者`6gb`
距离| 距离的支撑单元是`mi` 几英里，`yd` 对于院子，`ft` 对于脚，`in` 对于英寸，`km` 对于公里，`m` 对于米，`cm` 用于厘米，`mm` 用于毫米，`nmi` 或者`NM` 用于航海里程。| `5mi` 或者`4ft`
无单位的数量| 对于没有单位的大价值观，请使用`k` 对于Kilo，`m` 对于巨人，`g` 对于giga，`t` 对于tera，`p` 对于PETA。| `5k` 5,000

将输出单位转换为人类-可读的值，请参阅[常见的休息参数]({{site.url}}{{site.baseurl}}/opensearch/common-parameters/)。


---
layout: default
title: 命令
parent: PPL &ndash; 管道处理语言
grand_parent: SQL和PPL
nav_order: 2
redirect_from:
 - /search-plugins/ppl/commands/
---

# 命令

`PPL` 支持一切[`SQL` 常见的]({{site.url}}{{site.baseurl}}/search-plugins/sql/functions/) 功能，包括[相关搜索]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text/)，但也引入了更多功能（称为`commands`）可用`PPL` 仅有的。

## DEDUP

这`dedup` （数据重复数据删除）命令从搜索结果中删除字段定义的重复文档。

### 句法

```sql
dedup [int] <field-list> [keepempty=<bool>] [consecutive=<bool>]
```

场地| 描述| 类型| 必需的| 默认
：--- | ：--- |：--- |：--- |：---
`int` |  保留每种组合的指定数量的重复事件。数字必须大于0。如果您不指定数字，则仅保留第一个发生的事件，并且所有其他重复项都从结果中删除。| `string` | 不| 1
`keepempty` | 如果为true，请保留文档，如果字段列表中的任何字段具有空值或缺少字段。| `nested list of objects` | 不| 错误的
`consecutive` | 如果为true，则仅删除具有重复组合的连续事件。| `Boolean` | 不| 错误的
`field-list` | 指定逗号-界定字段列表。至少需要一个字段。| `String` 或逗号-分开的字符串清单| 是的| -

**示例1：在一个字段中删除**

删除具有相同性别的重复文档：

```sql
search source=accounts | dedup gender | fields account_number, gender;
```

| 帐号| 性别
：--- | ：--- |
1| m
13| F


**示例2：保留两个重复文档**

要保留两个重复的文档，具有相同的性别：

```sql
search source=accounts | dedup 2 gender | fields account_number, gender;
```

| 帐号| 性别
：--- | ：--- |
1| m
6| m
13| F

**示例3：默认情况下保留或忽略空字段**

保留两个重复文档`null` 现场值：

```sql
search source=accounts | dedup email keepempty=true | fields account_number, email;
```

| 帐号| 电子邮件
：--- | ：--- |
1| amberduke@pyrami.com
6| hattiebond@netagy.com
13| 无效的
18| daleadams@boink.com

删除重复文档`null` 现场值：

```sql
search source=accounts | dedup email | fields account_number, email;
```

| 帐号| 电子邮件
：--- | ：--- |
1| amberduke@pyrami.com
6| hattiebond@netagy.com
18| daleadams@boink.com

**示例4：连续文档的删除**

删除连续文档的重复物：

```sql
search source=accounts | dedup gender consecutive=true | fields account_number, gender;
```

| 帐号| 性别
：--- | ：--- |
1| m
13| F
18| m

### 限制

这`dedup` 命令未重写为OpenSearch DSL，仅在协调节点上执行。

## 评估

这`eval` 命令评估表达式并将其结果附加到搜索结果中。

### 句法

```sql
eval <field>=<expression> ["," <field>=<expression> ]...
```

场地| 描述| 必需的
：--- | ：--- |：---
`field` | 如果不存在字段名称，则添加一个新字段。如果字段名称已经存在，则将其覆盖。| 是的
`expression` | 指定任何支持的表达式。| 是的

**示例1：创建一个新字段**

创建一个新的`doubleAge` 每个文档的字段。`doubleAge` 是`age` 乘以2：

```sql
search source=accounts | eval doubleAge = age * 2 | fields age, doubleAge;
```

| 年龄| 双打
：--- | ：--- |
32| 64
36| 72
28| 56
33| 66

*示例2*：覆盖现有字段

覆盖`age` 字段与`age` 加1：

```sql
search source=accounts | eval age = age + 1 | fields age;
```

| 年龄
：--- |
| 33
| 37
| 29
| 34

**示例3：创建一个新的字段，该字段定义为`eval` 命令**

创建一个新领域`ddAge`。`ddAge` 是`doubleAge` 乘以2，其中`doubleAge` 在`eval` 命令：

```sql
search source=accounts | eval doubleAge = age * 2, ddAge = doubleAge * 2 | fields age, doubleAge, ddAge;
```

| 年龄| 双打| ddage
：--- | ：--- |
| 32| 64| 128
| 36| 72| 144
| 28| 56| 112
| 33| 66| 132


### 局限性

这``评估`` 命令未重写为OpenSearch DSL，仅在协调节点上执行。

## 字段

使用`fields` 命令保留或删除搜索结果中的字段。

### 句法

```sql
fields [+|-] <field-list>
```

场地| 描述| 必需的| 默认
：--- | ：--- |：---|：---
`index` | 加上（+）仅保留字段列表中指定的字段。减 （-）删除字段列表中指定的所有字段。| 不| +
`field list` | 指定逗号-界定字段列表。| 是的| 没有默认值

**示例1：从结果中选择指定的字段**

要得到`account_number`，，，，`firstname`， 和`lastname` 搜索结果的字段：

```sql
search source=accounts | fields account_number, firstname, lastname;
```

| 帐号| 名| 姓
：--- | ：--- |
| 1| 琥珀色| 公爵
| 6| 哈蒂| 纽带
| 13| Nanette| 贝茨
| 18| 戴尔| 亚当斯

**示例2：从搜索结果中删除指定的字段**

删除`account_number` 搜索结果的字段：

```sql
search source=accounts | fields account_number, firstname, lastname | fields - account_number;
```

| 名| 姓
：--- | ：--- |
| 琥珀色| 公爵
| 哈蒂| 纽带
| Nanette| 贝茨
| 戴尔| 亚当斯


## 解析

使用`parse` 命令使用正则表达式解析文本字段，并将结果附加到搜索结果。

### 句法

```sql
parse <field> <regular-expression>
```

场地| 描述| 必需的
：--- | ：--- |：---
场地| 文本字段。| 是的
常规的-表达| 正则表达式用于从给定的测试字段提取新字段。如果存在新字段名称，它将替换原始字段。| 是的

正则表达式用于将每个文档的整个文本字段与Java Regex引擎匹配。表达式中的每个命名捕获组都将成为一个新的``细绳`` 场地。

**示例1：创建新字段**

该示例显示了如何创建新字段`host` 对于每个文档。`host` 将是主机名之后`@` 在`email` 场地。解析空字段将返回一个空字符串。

```sql
os> source=accounts | parse email '.+@(?<host>.+)' | fields email, host ;
fetched rows / total rows = 4/4
```

| 电子邮件| 主持人
：--- | ：--- |
| amberduke@pyrami.com| pyrami.com
| hattiebond@netagy.com| netagy.com
| 无效的| 无效的
| daleadams@boink.com| boink.com

*示例2*：覆盖现有字段

该示例显示了如何以删除街道号码覆盖现有地址字段。

```sql
os> source=accounts | parse address '\d+ (?<address>.+)' | fields address ;
fetched rows / total rows = 4/4
```

| 地址
：--- |
| 福尔摩斯巷
| 布里斯托尔街
| 麦迪逊街
| 哈钦森法院

**示例3：过滤和排序被铸造出分析字段**

该示例显示了如何在地址字段中排序高于500的街道号码。

```sql
os> source=accounts | parse address '(?<streetNumber>\d+) (?<street>.+)' | where cast(streetNumber as int) > 500 | sort num(streetNumber) | fields streetNumber, street ;
fetched rows / total rows = 3/3
```

| 街牌号码| 街道
：--- | ：--- |
| 671| 布里斯托尔街
| 789| 麦迪逊街
| 880| 福尔摩斯巷

### 限制

使用parse命令时存在一些限制：

- 不能再次解析由解析定义的字段。例如，`source=accounts | parse address '\d+ (?<street>.+)' | parse street '\w+ (?<road>\w+)' ;` 将无法返回任何表达式。
- 其他命令不能覆盖由解析定义的字段。例如，输入`source=accounts | parse address '\d+ (?<street>.+)' | eval street='1' | where street='1' ;` `where` 因为`street` 不能被覆盖。
- 解析使用的文本字段不能被覆盖。例如，输入`source=accounts | parse address '\d+ (?<street>.+)' | eval address='1' ;` `street` 由于地址被覆盖，因此不会是解析。
- 在使用解析定义的字段之后，无法过滤/分类。`stats` 命令。例如，`source=accounts | parse email '.+@(?<host>.+)' | stats avg(age) by host | where host=pyrami.com ;` `where` 不会解析列出的域。

## 改名

使用`rename` 命令在搜索结果中重命名一个或多个字段。

### 句法

```sql
rename <source-field> AS <target-field>["," <source-field> AS <target-field>]...
```

场地| 描述| 必需的
：--- | ：--- |：---
`source-field` | 您要重命名的字段名称。| 是的
`target-field` | 您要重命名的名称。| 是的

**示例1：重命名一个字段**

重命名`account_number` 字段为`an`：

```sql
search source=accounts | rename account_number as an | fields an;
```

| 一个
：--- |
| 1
| 6
| 13
| 18

**示例2：重命名多个字段**

重命名`account_number` 字段为`an` 和`employer` 作为`emp`：

```sql
search source=accounts | rename account_number as an, employer as emp | fields an, emp;
```

| 一个| emp
：--- | ：--- |
| 1| Pyrami
| 6| 网络
| 13| 裁员
| 18| 无效的

### 限制

这`rename` 命令未重写为OpenSearch DSL，仅在协调节点上执行。

## 种类

使用`sort` 命令按指定字段对搜索结果进行排序。

### 句法

```sql
sort [count] <[+|-] sort-field>...
```

场地| 描述| 必需的| 默认
：--- | ：--- |：---
`count` | 从排序结果返回的最大数字结果。如果计数= 0，则所有结果将返回。| 不| 1000
`[+|-]` | 使用加上[+]按升序和减去[-]按降序排序。| 不| 上升顺序
`sort-field` | 指定您要排序的字段。| 是的| -

**示例1：按一个字段进行排序**

对所有文档进行排序`age` 按顺序排列的字段：

```sql
search source=accounts | sort age | fields account_number, age;
```

| 帐号| 年龄|
：--- | ：--- |
| 13| 28
| 1| 32
| 18| 33
| 6| 36

**示例2：按一个字段排序并返回所有结果**

对所有文档进行排序`age` 按顺序排列的字段并将计数指定为0，以恢复所有结果：

```sql
search source=accounts | sort 0 age | fields account_number, age;
```

| 帐号| 年龄|
：--- | ：--- |
| 13| 28
| 1| 32
| 18| 33
| 6| 36

**示例3：按降序按一个字段进行排序**

对所有文档进行排序`age` 下降顺序的字段：

```sql
search source=accounts | sort - age | fields account_number, age;
```

| 帐号| 年龄|
：--- | ：--- |
| 6| 36
| 18| 33
| 1| 32
| 13| 28

**示例4：指定要返回的分类文档的数量**

对所有文档进行排序`age` 按顺序排列的字段并将数字指定为2，以恢复两个结果：

```sql
search source=accounts | sort 2 age | fields account_number, age;
```

| 帐号| 年龄|
：--- | ：--- |
| 13| 28
| 1| 32

**示例5：按多个字段进行排序**

对所有文档进行排序`gender` 按顺序排列和`age` 下降顺序的字段：

```sql
search source=accounts | sort + gender, - age | fields account_number, gender, age;
```

| 帐号| 性别| 年龄|
：--- | ：--- | ：--- |
| 13| F| 28
| 6| m| 36
| 18| m| 33
| 1| m| 32

## 统计

使用`stats` 命令从搜索结果中汇总。

下表列出了聚合功能，还指示每个人如何处理null或缺失值：

功能| 无效的| 丢失的
：--- | ：--- |：---
`COUNT` | 不算| 不算
`SUM` | 忽略| 忽略
`AVG` | 忽略| 忽略
`MAX` | 忽略| 忽略
`MIN` | 忽略| 忽略


### 句法

```
stats <aggregation>... [by-clause]...
```

场地| 描述| 必需的| 默认
：--- | ：--- |：---
`aggregation` | 指定统计聚合函数。此功能的参数必须是一个字段。| 是的| 1000
`by-clause` | 指定一个或多个字段以按结果进行分组。如果未指定，`stats` 命令仅返回一行，这是整个结果集中的聚合。| 不| -

**示例1：计算字段的平均值**

计算平均`age` 在所有文件中：

```sql
search source=accounts | stats avg(age);
```

| AVG（年龄）
：--- |
| 32.25

**示例2：按组计算字段的平均值**

计算性别分组的平均年龄：

```sql
search source=accounts | stats avg(age) by gender;
```

| 性别| AVG（年龄）
：--- | ：--- |
| F| 28.0
| m| 33.66666666666664

**示例3：按组计算字段的平均值和总和**

计算性别分组年龄的平均和总和：

```sql
search source=accounts | stats avg(age), sum(age) by gender;
```

| 性别| AVG（年龄）| 总和（年龄）
：--- | ：--- |
| F| 28| 28
| m| 33.66666666666664| 101

**示例4：计算字段的最大值**

计算最大年龄：

```sql
search source=accounts | stats max(age);
```

| 最大（年龄）
：--- |
| 36

**示例5：按组计算字段的最大值和最小值**

计算性别分组的最大和最低年龄值：

```sql
search source=accounts | stats max(age), min(age) by gender;
```

| 性别| 最小（年龄）| 最大（年龄）
：--- | ：--- | ：--- |
| F| 28| 28
| m| 32| 36

## 在哪里

使用`where` 用bool表达式命令以过滤搜索结果。这`where` 命令仅在布尔表达式评估为true时才返回结果。

### 句法

```sql
where <boolean-expression>
```

场地| 描述| 必需的
：--- | ：--- |：---
`bool-expression` | 评估布尔值的表达式。| 不

**示例：带有条件的过滤结果设置**

从`accounts` 索引在哪里`account_number` 是1或性别是`F`：

```sql
search source=accounts | where account_number=1 or gender=\"F\" | fields account_number, gender;
```

| 帐号| 性别
：--- | ：--- |
| 1| m
| 13| F

## 头

使用`head` 命令在指定的搜索顺序中返回第一个n数的结果。

### 句法

```sql
head [N]
```

场地| 描述| 必需的| 默认
：--- | ：--- |：---
`N` | 指定要返回的结果数。| 不| 10

**示例1：获取前10个结果**

要获得前10个结果：

```sql
search source=accounts | fields firstname, age | head;
```

| 名| 年龄
：--- | ：--- |
| 琥珀色| 32
| 哈蒂| 36
| Nanette| 28

**示例2：获取第一个N结果**

要获得前两个结果：

```sql
search source=accounts | fields firstname, age | head 2;
```

| 名| 年龄
：--- | ：--- |
| 琥珀色| 32
| 哈蒂| 36

### 限制

这`head` 命令未重写为OpenSearch DSL，仅在协调节点上执行。

## 稀有的

使用`rare` 命令在字段列表中找到所有字段的最小值。
对于组的每组值，最多可返回10个结果-通过字段。

### 句法

```sql
rare <field-list> [by-clause]
```

场地| 描述| 必需的
：--- | ：--- |：---
`field-list` | 指定逗号-字段名称的界定列表。| 不
`by-clause` | 指定一个或多个字段以按结果进行分组。| 不

**示例1：在字段中找到最小的共同值**

找到性别最不常见的价值观：

```sql
search source=accounts | rare gender;
```

| 性别
：--- |
| F
| m

**示例2：找到由性别分组的最小共同值**

找到最不常见的年龄分组的性别：

```sql
search source=accounts | rare age by gender;
```

| 性别| 年龄
：--- | ：--- |
| F| 28
| m| 32
| m| 33

### 限制

这`rare` 命令未重写为OpenSearch DSL，仅在协调节点上执行。

## 顶部 {#顶部-命令}

使用`top` 命令在字段列表中找到所有字段的最常见值。

### 句法

```sql
top [N] <field-list> [by-clause]
```

场地| 描述| 默认
：--- | ：--- |：---
`N` | 指定要返回的结果数。| 10
`field-list` | 指定逗号-字段名称的界定列表。| -
`by-clause` | 指定一个或多个字段以按结果进行分组。| -

**示例1：在字段中找到最常见的值**

找到最常见的性别：

```sql
search source=accounts | top gender;
```

| 性别
：--- |
| m
| F

**示例2：在字段中找到最常见的值**

找到最常见的性别：

```sql
search source=accounts | top 1 gender;
```

| 性别
：--- |
| m

**示例3：找到由性别分组的最常见值**

找到由性别分组的最常见的年龄：

```sql
search source=accounts | top 1 age by gender;
```

| 性别| 年龄
：--- | ：--- |
| F| 28
| m| 32

### 限制

这`top` 命令未重写为OpenSearch DSL，仅在协调节点上执行。

## 广告

这`ad` 命令应用随机切割森林（RCF）算法[ML Commons插件]({{site.url}}{{site.baseurl}}/ml-commons-plugin/index/) 在搜索结果上由ppl命令返回。基于输入，该插件使用两种类型的RCF算法：固定时间RCF处理时间-用于处理非处理的串联数据和批量RCF-时间-系列数据。

### 语法：时间固定在时间上-系列数据命令

```sql
ad <shingle_size> <time_decay> <time_field>
```

场地| 描述| 必需的
：--- | ：--- |：---
`shingle_size` | 最新记录的连续序列。默认值为8。| 不
`time_decay` | 指定在计算异常分数时要考虑的过去的多少。默认值为0.001。| 不
`time_field` | 指定RCF用作时间的时间-系列数据。必须是一个长的值，例如Miliseconds中的时间戳，或者是字符串值"yyyy-MM-dd HH:mm:ss"。| 是的

### 语法：非批次RCF-时间-系列数据命令

```sql
ad <shingle_size> <time_decay>
```

场地| 描述| 必需的
：--- | ：--- |：---
`shingle_size` | 最新记录的连续序列。默认值为8。| 不
`time_decay` | 指定在计算异常分数时要考虑的过去的多少。默认值为0.001。| 不

**示例1：随着时间的流逝，从出租车乘车数据中检测纽约市的事件-系列数据**

该示例训练RCF模型，并使用该模型在时间内检测异常-系列乘车数据。

PPL查询：

```sql
os> source=nyc_taxi | fields value, timestamp | AD time_field='timestamp' | where value=10844.0
```

价值| 时间戳| 分数| Anomaly_grade
：--- | ：--- |：--- | ：---
10844.0| 1404172800000| 0.0| 0.0

**示例2：通过非出租车乘车数据检测纽约市的事件-时间-系列数据**

PPL查询：

```sql
os> source=nyc_taxi | fields value | AD | where value=10844.0
```

价值| 分数| 异常
：--- | ：--- |：--- 
| 10844.0| 0.0| 错误的

## Kmeans

KMeans命令将ML Commons插件的KMeans算法应用于提供的PPL命令的搜索结果。

### 句法

```sql
kmeans <cluster-number>
```

为了`cluster-number`，输入要将数据点分组到的群集数量。

**示例：组虹膜数据**

该示例显示了如何根据每个样品测得的四个特征的组合来对三种虹膜物种（虹膜，虹膜弗吉尼亚和艾里斯·versicolor）进行分类：萼片和花瓣的长度和宽度。

PPL查询：

```sql
os> source=iris_data | fields sepal_length_in_cm, sepal_width_in_cm, petal_length_in_cm, petal_width_in_cm | kmeans 3
```

sepal_length_in_cm| sepal_width_in_cm| petal_length_in_cm| petal_width_in_cm| 聚类
：--- | ：--- |：--- | ：--- | ：--- 
| 5.1| 3.5| 1.4| 0.2| 1
| 5.6| 3.0| 4.1| 1.3| 0
| 6.7| 2.5| 5.8| 1.8| 2


---
layout: default
title: 令牌过滤器
nav_order: 70
has_children: true
has_toc: false
---

# 令牌过滤器

令牌过滤器从令牌仪接收令牌流，然后添加，删除或修改令牌。例如，令牌过滤器可能会降低令牌，以便`Actions` 变成`action`，删除像`than`，或添加同义词`talk` 对于这个词`speak`。

下表列出了OpenSearch支持的所有令牌过滤器。

令牌过滤器| 下面的露烯代币过滤器|  描述
`apostrophe` | [脱发器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/tr/ApostropheFilter.html) | 在每个包含撇号的令牌中，`apostrophe` 令牌过滤器删除了撇号本身和撇号后的所有字符。
`asciifolding` | [AsciifoldingFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html) | 转换字母，数字和符号字符。
`cjk_bigram` | [cjkbigramfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/cjk/CJKBigramFilter.html) | 形成中国，日本和韩国（CJK）代币的大型大型。
`cjk_width` | [cjkwidthfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html) | 根据以下规则将中国，日语和韩语（CJK）令牌归一化：<br>- 折叠满-宽度ASCII字符变体中等效的基本拉丁字符。<br>- 折叠一半-宽度katakana角色变体成相同的卡纳角色。
`classic` | [Classicfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/standard/ClassicFilter.html) | 执行可选帖子-在经典令牌生成的令牌上处理。去除所有占有人（`'s`）并去除`.` 从缩写词。
`common_grams` | [CONMONGRAMSFILTER](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/commongrams/CommonGramsFilter.html) | 生成bigrams，以列出经常出现的术语列表。输出既包含单个术语又包含bigrams。
`conditional` | [有条件的腹部](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ConditionalTokenFilter.html) | 将令牌过滤器的有序列表应用于与脚本中提供的条件相匹配的令牌。
`decimal_digit` | [demimaldigitfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/core/DecimalDigitFilter.html) | 将Unicode十进制数字一般类别中的所有数字转换为基本拉丁数字（0--9）。
`delimited_payload` | [DelemitedPayloadTokenFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/payloads/DelimitedPayloadTokenFilter.html) | 根据提供的定界符将令牌流分隔为具有相应有效载荷的令牌。代币由定界符之前的所有字符组成，有效载荷由定界符之后的所有字符组成。例如，如果定界符是`|`，然后对于字符串`foo|bar`，`foo` 是令牌和`bar` 是有效载荷。
[`delimited_term_freq`]({{site.url}}{{site.baseurl}}/analyzers/token-filters/delimited-term-frequency/) | [划界术语](https://lucene.apache.org/core/9_7_0/analysis/common/org/apache/lucene/analysis/miscellaneous/DelimitedTermFrequencyTokenFilter.html) | 根据提供的定界符，将令牌流分隔为具有相应项频率的令牌。一个令牌由定界符之前的所有字符组成，术语频率是定界符之后的整数。例如，如果定界符是`|`，然后对于字符串`foo|5`，`foo` 是令牌和`5` 是术语频率。
`dictionary_decompounder` | [字典COMPOUNDWORDTOKENFILTER](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/compound/DictionaryCompoundWordTokenFilter.html) | 分解许多日耳曼语中发现的复合词。
`edge_ngram` | [EdgengramTokenFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenFilter.html) | 将给定令牌归为toge Edge n-克（n-从代币开始时开始的克`min_gram` 和`max_gram`。可选，保留原始令牌。
`elision` | [ElisionFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/util/ElisionFilter.html) | 删除指定的[精力](https://en.wikipedia.org/wiki/Elision) 从代币的开始。例如，更改`l'avion` （飞机）`avion` （飞机）。
`fingerprint` | [指纹膜](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/FingerprintFilter.html) | 对令牌列表进行分类并重复变性，并将令牌串联成一个令牌。
`flatten_graph` | [FlattenGraphFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/core/FlattenGraphFilter.html) | 图形令牌滤波器产生的令牌图，例如`synonym_graph` 或者`word_delimiter_graph`，使图适用于索引。
`hunspell` | [Hunspellstemfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/hunspell/HunspellStemFilter.html) | 用途[亨斯佩尔](https://en.wikipedia.org/wiki/Hunspell) 茎令牌的规则。由于Hunspell支持一个具有多个词干的单词，因此该过滤器可以为每个消耗的令牌发出多个令牌。要求您配置一种或多种语言-特定的烟囱字典。
`hyphenation_decompounder` | [连字符COMPOUNDWORDTOKENFILTER](https://lucene.apache.org/core/9_8_0/analysis/common/org/apache/lucene/analysis/compound/HyphenationCompoundWordTokenFilter.html) | 使用XML-基于连字符模式以复合单词查找潜在子词，并根据指定的单词列表检查子字。令牌输出仅包含单词列表中的子字。
`keep_types` | [TypeTokenFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/core/TypeTokenFilter.html) | 保留或去除特定类型的令牌。
`keep_word` | [保持Wordfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeepWordFilter.html) | 根据指定的单词列表检查令牌，并仅保留列表中的单词列表。
`keyword_marker` | [关键字标记物](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordMarkerFilter.html) | 标记将令牌指定为关键字，从而阻止它们被茎。
`keyword_repeat` | [keywordrepeatFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordRepeatFilter.html) | 两次发出每个传入的令牌：一次是关键字，一次是一个非关键字-关键词。
`kstem` | [Kstemfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/en/KStemFilter.html) | 提供KETSTEM-基于英语的基础。结合算法的算法与内置的-在字典中。
`length` | [长度滤波器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/LengthFilter.html) | 删除其长度短或更长的令牌`min` 和`max`。
`limit` | [Limittokencountfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/LimitTokenCountFilter.html) | 限制输出令牌的数量。一个常见的用例是限制基于令牌计数的文档字段值的大小。
`lowercase` | [LowercaseFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html) | 将令牌转换为小写。默认值[LowercaseFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html) 是针对英语。您可以设置`language` 参数为`greek` （用途[希腊语cassisfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/el/GreekLowerCaseFilter.html)），`irish` （用途[伊利兰语cassisfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ga/IrishLowerCaseFilter.html)）， 或者`turkish` （用途[turkishlowercasefilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/tr/TurkishLowerCaseFilter.html)）。
`min_hash` | [Minhashfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/minhash/MinHashFilter.html) | 使用[Minhash技术](https://en.wikipedia.org/wiki/MinHash) 估计文件相似性。依次在令牌流上执行以下操作：<br> 1.在流中每个令牌。<br> 2.将哈希分配给桶，仅保留每个桶的最小哈希。<br> 3.将每个存储桶的最小哈希作为令牌流输出。
`multiplexer` | N/A。| 在同一位置发出多个令牌。通过每个指定的过滤器列表分别运行每个令牌，并将结果作为单独的令牌输出。
`ngram` | [ngramTokenFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenFilter.html) | 将给定的令牌归为n-克长度之间的克`min_gram` 和`max_gram`。
正常化| `arabic_normalization`：[阿拉伯式化合物](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizer.html) <br>`german_normalization`：[生物化滤光器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html) <br>`hindi_normalization`：[后施剂](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizer.html) <br>`indic_normalization`：[指示剂](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizer.html) <br>`sorani_normalization`：[丝氨酸分子](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizer.html) <br>`persian_normalization`：[Persiannormalizer](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizer.html) <br>`scandinavian_normalization` ：[斯堪的纳维亚法性滤光器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html) <br>`scandinavian_folding`：[斯堪的纳维亚折叠式](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html) <br>`serbian_normalization`：[塞尔维亚法性滤光器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/sr/SerbianNormalizationFilter.html) | 将列出的语言之一的字符归一化。
`pattern_capture` | N/A。| 为提供的正则表达式中的每个捕获组生成一个令牌。用途[Java正则表达语法](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)。
`pattern_replace` | N/A。| 匹配提供的正则表达式中的模式，并替换匹配的子字符串。用途[Java正则表达语法](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)。
`phonetic` | N/A。| 使用语音编码器为代币流中的每个令牌发出形态令牌。需要安装`analysis-phonetic` 插入。
`porter_stem` | [Porterstemfilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/en/PorterStemFilter.html) | 使用[搬运工算法](https://tartarus.org/martin/PorterStemmer/) 为英语执行算法。
`predicate_token_filter` | N/A。| 删除与指定的谓词脚本不匹配的令牌。仅支持内联无痛脚本。
`remove_duplicates` | [删除了造成的](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/RemoveDuplicatesTokenFilter.html) | 删除处于相同位置的重复令牌。
`reverse` | [反向滤波器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/reverse/ReverseStringFilter.html) | 逆转与令牌流中每个令牌相对应的字符串。例如，令牌`dog` 变成`god`。
`shingle` | [Shinglefilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/shingle/ShingleFilter.html) | 产生长度的带状疱疹`min_shingle_size` 和`max_shingle_size` 对于令牌流中的令牌。带状疱疹类似于n-克，但适用于单词而不是字母。例如，两个-单词带状的单词添加到umigram的列表中[`contribute`，`to`，`opensearch`] 是 [`contribute to`，`to opensearch`]。
`snowball` | N/A。| 使用一个[滚雪球-生成的茎](https://snowballstem.org/)。您可以使用`snowball` 用以下语言在`language` 场地：`Arabic`，`Armenian`，`Basque`，`Catalan`，`Danish`，`Dutch`，`English`，`Estonian`，`Finnish`，`French`，`German`，`German2`，`Hungarian`，`Irish`，`Italian`，`Kp`，`Lithuanian`，`Lovins`，`Norwegian`，`Porter`，`Portuguese`，`Romanian`，`Russian`，`Spanish`，`Swedish`，`Turkish`。
`stemmer` | N/A。| 为以下语言提供算法`language` 场地：`arabic`，`armenian`，`basque`，`bengali`，`brazilian`，`bulgarian`，`catalan`，`czech`，`danish`，`dutch`，`dutch_kp`，`english`，`light_english`，`lovins`，`minimal_english`，`porter2`，`possessive_english`，`estonian`，`finnish`，`light_finnish`，`french`，`light_french`，`minimal_french`，`galician`，`minimal_galician`，`german`，`german2`，`light_german`，`minimal_german`，`greek`，`hindi`，`hungarian`，`light_hungarian`，`indonesian`，`irish`，`italian`，`light_italian`，`latvian`，`Lithuanian`，`norwegian`，`light_norwegian`，`minimal_norwegian`，`light_nynorsk`，`minimal_nynorsk`，`portuguese`，`light_portuguese`，`minimal_portuguese`，`portuguese_rslp`，`romanian`，`russian`，`light_russian`，`sorani`，`spanish`，`light_spanish`，`swedish`，`light_swedish`，`turkish`。
`stemmer_override` | N/A。| 通过应用自定义映射来覆盖构造算法，以使所提供的条款不会被阻止。
`stop` | [停止滤器](https://lucene.apache.org/core/8_7_0/core/org/apache/lucene/analysis/StopFilter.html) | 从令牌流中删除停止单词。
`synonym` | N/A。| 提供分析过程的同义词列表。使用配置文件提供同义词列表。
`synonym_graph` | N/A。| 为分析过程提供同义词列表，包括多字形同义词。
`trim` | [修剪器](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/TrimFilter.html) | 将每个令牌从溪流中的每个令牌带领先和尾随的空间。
`truncate` | [TruncateTokenFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/TruncateTokenFilter.html) | 长度超过指定字符限制的截短令牌。
`unique` | N/A。| 通过从流中删除复制令牌，确保每个令牌都是唯一的。
`uppercase` | [高层](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html) | 将令牌转换为大写。
`word_delimiter` | [WordDelimiterFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/WordDelimiterFilter.html) | 分裂令牌-字母数字字符并根据指定规则执行归一化。
`word_delimiter_graph` | [WordDelimiterGraphFilter](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/WordDelimiterGraphFilter.html) | 分裂令牌-字母数字字符并根据指定规则执行归一化。分配多人-位置令牌a`positionLength` 属性。


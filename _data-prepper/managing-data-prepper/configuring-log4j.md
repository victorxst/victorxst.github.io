---
layout: default
title: 配置log4j
parent: 管理数据预先
nav_order: 20
---

# 配置log4j

您可以在Data Prepper中使用Log4J配置日志记录。

## 记录

数据预先使用[slf4j](https://www.slf4j.org/) 与[log4j 2绑定](https://logging.apache.org/log4j/2.x/log4j-slf4j-impl.html)。

对于Data Prepper版本2.0及以后，可以在`config/log4j2.properties` 在应用程序的主目录中。log4j 2的默认属性可以在`log4j2-rolling.properties` 在 *共享-config*目录。

对于2.0之前的Data Prepper版本，可以通过设置log4j 2配置文件来覆盖`log4j.configurationFile` 运行数据预先运行时的系统属性。log4j 2的默认属性可以在`log4j2.properties` 在 *共享-config*目录。

### 例子

在运行数据预先计算机时，可以通过设置系统属性来覆盖以下命令`-Dlog4j.configurationFile={property_value}`， 在哪里`{property_value}` 是Log4J 2配置文件的路径：

```
java "-Dlog4j.configurationFile=config/custom-log4j2.properties" -jar data-prepper-core-$VERSION.jar pipelines.yaml data-prepper-config.yaml
```

看到[log4j 2配置文档](https://logging.apache.org/log4j/2.x/manual/configuration.html) 有关Log4J 2配置的更多信息。



---
layout: default
title: 管道 
parent: 下沉
grand_parent: 管道
nav_order: 55
---

# 管道

使用`pipeline` 下沉以写入另一个管道。

## 配置选项

这`pipeline` 接收器支持以下配置选项。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
姓名| 是的| 细绳| 要写的管道的名称。

## 用法

以下示例配置了`pipeline` 写信给一个名字的管道的水槽`movies`：

```
sample-pipeline:
  sink:
    - pipeline:
        name: movies
```


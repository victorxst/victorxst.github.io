---
layout: default
title: 文件
parent: 下沉
grand_parent: 管道
nav_order: 45
---

# 文件

使用`file` 下沉以创建一个平坦的文件输出，通常`.log` 文件。

## 配置选项

下表描述了您可以配置的选项`file` 下沉。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
小路| 是的| 细绳| 输出文件的路径（例如`logs/my-transformed-log.log`）。

## 用法

以下示例显示了的基本用法`file` 下沉：

```
sample-pipeline:
  sink:
    - file:
        path: path/to/output-file
```



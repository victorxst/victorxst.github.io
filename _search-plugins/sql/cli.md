---
layout: default
title: SQL和PPL CLI
parent: SQL和PPL
nav_order: 3
redirect_from:
 - /search-plugins/sql/cli/
---

# SQL和PPL CLI

SQL和PPL命令行接口（CLI）是一个独立的Python应用程序，您可以使用该应用程序启动`opensearchsql` 命令。

 要使用SQL和PPL CLI，请在OpenSearch实例上安装SQL插件，使用MACOS或Linux运行CLI，然后连接到任何有效的OpenSearch Endpoint。

![SQL CLI]({{site.url}}{{site.baseurl}}/images/cli.gif)

## 特征

SQL和PPL CLI具有以下功能：

- 多-线输入
- PPL支持
- SQL语法和索引名称的自动完成
- 语法突出显示
- 格式化输出：
  - 表格格式
  - 带有颜色的字段名称
  - 启用了水平显示（默认情况下）和垂直显示，当输出太宽而无法实现您的终端，以获得更好的可视化
  - 大型输出的分页
- 使用或不启用安全性的工作
- 支持加载配置文件
- 支持所有SQL插件查询

## 安装

启动您的本地OpenSearch实例，并确保已安装SQL插件。

1. 安装CLI：
```console
pip3 install opensearchsql
```

SQL CLI仅与Python 3一起使用。
{: .note }

2. 要启动CLI，请运行：
```console
opensearchsql https://localhost:9200 --username admin --password admin
```
默认情况下，`opensearchsql` 命令连接到http：// localhost：9200。

## 配置

首次启动SQL CLI时，将自动创建一个配置文件`~/.config/opensearchsql-cli/config` （对于MacOS和Linux），配置为自动-此后加载。

您可以配置以下连接属性：

- `endpoint`：您无需指定选项。遵循启动命令的任何内容`opensearchsql` 被认为是终点。如果您不提供端点，则默认情况下，SQL CLI连接到http：// localhost：9200。
- `-u/-w`：支持HTTP基本身份验证的用户名和密码，例如使用安全插件或罚款-Amazon OpenSearch服务的粒度访问控制。
- `--aws-auth`：打开AWS SIGV4身份验证以连接到Amazon OpenSearch端点。与AWS CLI一起使用（`aws configure`）检索本地AWS配置以进行身份验证和连接。

有关所有可用配置的列表，请参见[clirc](https://github.com/opensearch-project/sql/blob/1.x/sql-cli/src/opensearch_sql_cli/conf/clirc)。

## 使用CLI

1. 运行CLI工具。如果您的群集使用默认的安全设置运行，请使用以下命令：
```console
opensearchsql --username admin --password admin https://localhost:9200
```
如果您的群集在没有安全性的情况下运行，请运行：
```console
opensearchsql
```

2. 运行示例SQL命令：
```sql
SELECT * FROM accounts;
```

默认情况下，您会看到最大输出为200行。要显示更多结果，请添加一个`LIMIT` 带有所需值的子句。

要退出CLI工具，请选择**Ctrl+D。**。
{: .tip }

## 将CLI与PPL一起使用

1. 通过指定查询语言来运行CLI：
```console
opensearchsql -l ppl <params>
```

2. 执行PPL查询：
```sql
source=accounts | fields firstname, lastname
```

## 查询选项

使用以下命令行选项运行单个查询：

- `-q`：跟随一个查询
- `-f`：指定JDBC或原始格式输出
- `-v`：垂直显示数据
- `-e`：将SQL转换为DSL

## CLI选项

- `--help`：帮助页面以获取选项
- `-l`：查询语言选项。可用选项是`sql` 和`ppl`。默认为`sql`
- `-p`：始终使用Pager显示输出
- `--clirc`：提供配置文件的路径


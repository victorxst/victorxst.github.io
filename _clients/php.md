---
layout: default
title: PHP客户端
nav_order: 70
---

# PHP客户端

OpenSearch PHP客户端提供了一种与OpenSearch集群交互的更安全，更简单的方法。您可以构建一个openSearch客户端，该客户端将请求将请求发送到群集。客户端包含一个API库，可让您在群集上执行不同的操作并返回标准响应主体。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-php回购](https://github.com/opensearch-project/opensearch-php)。

## 设置

要将客户端添加到您的项目中，请使用[作曲家](https://getcomposer.org/)：

```bash
composer require opensearch-project/opensearch-php
```
{％include copy.html％}

要安装客户端的特定主要版本，请运行以下命令：

```bash
composer require opensearch-project/opensearch-php:<version>
```
{％include copy.html％}

然后在您的代码中需要从作曲家的自动加载文件：

```php
require __DIR__ . '/vendor/autoload.php';
```
{％include copy.html％}

## 连接到OpenSearch

要连接到默认的OpenSearch主机，请使用地址创建客户端对象`https://localhost:9200` 如果您使用的是安全插件：

```php
$client = (new \OpenSearch\ClientBuilder())
    ->setHosts(['https://localhost:9200'])
    ->setBasicAuthentication('admin', 'admin') // For testing only. Don't store credentials in code.
    ->setSSLVerification(false) // For testing only. Use certificate for validation
    ->build();
```
{％include copy.html％}

## 连接到Amazon OpenSearch服务

下面的示例说明了连接到Amazon OpenSearch服务：

```php
$client = (new \OpenSearch\ClientBuilder())
    ->setSigV4Region('us-east-2')

    ->setSigV4Service('es')
    
    // Default credential provider.
    ->setSigV4CredentialProvider(true)
    
    // Using a custom access key and secret
    ->setSigV4CredentialProvider([
      'key' => 'awskeyid',
      'secret' => 'awssecretkey',
    ])
    
    ->build();
```
{％include copy.html％}

## 连接到Amazon OpenSearch无服务器

以下示例说明了连接到Amazon OpenSearch无服务器服务：

```php
$client = (new \OpenSearch\ClientBuilder())
    ->setSigV4Region('us-east-2')

    ->setSigV4Service('aoss')
    
    // Default credential provider.
    ->setSigV4CredentialProvider(true)
    
    // Using a custom access key and secret
    ->setSigV4CredentialProvider([
      'key' => 'awskeyid',
      'secret' => 'awssecretkey',
    ])
    
    ->build();
```
{％include copy.html％}


## 创建索引

要创建具有自定义设置的OpenSearch索引，请使用以下代码：

```php
$indexName = 'test-index-name';

// Create an index with non-default settings.
$client->indices()->create([
    'index' => $indexName,
    'body' => [
        'settings' => [
            'index' => [
                'number_of_shards' => 4
            ]
        ]
    ]
]);
```
{％include copy.html％}

## 索引文档

您可以使用以下代码将文档索引到OpenSearch：

```php
$client->create([
    'index' => $indexName,
    'id' => 1,
    'body' => [
        'title' => 'Moneyball',
        'director' => 'Bennett Miller',
        'year' => 2011
    ]
]);
```
{％include copy.html％}

## 搜索文档

以下代码使用`multi_match` 查询搜索"miller" 在标题和导演领域。它增加了文件"miller" 出现在标题字段中：

```php
var_dump(
    $client->search([
        'index' => $indexName,
        'body' => [
            'size' => 5,
            'query' => [
                'multi_match' => [
                    'query' => 'miller',
                    'fields' => ['title^2', 'director']
                ]
            ]
        ]
    ])
);
```
{％include copy.html％}

## 删除文档

您可以使用以下代码删除文档：

```php
$client->delete([
    'index' => $indexName,
    'id' => 1,
]);
```
{％include copy.html％}

## 删除索引

您可以使用以下代码删除索引：

```php
$client->indices()->delete([
    'index' => $indexName
]);
```
{％include copy.html％}

## 样本程序

以下示例程序创建了一个客户端，添加了一个非索引-默认设置，插入文档，搜索文档，删除文档，然后删除索引：

```php
<?php

require __DIR__ . '/vendor/autoload.php';

$client = (new \OpenSearch\ClientBuilder())
    ->setHosts(['https://localhost:9200'])
    ->setBasicAuthentication('admin', 'admin') // For testing only. Don't store credentials in code.
    ->setSSLVerification(false) // For testing only. Use certificate for validation
    ->build();

$indexName = 'test-index-name';

// Print OpenSearch version information on console.
var_dump($client->info());

// Create an index with non-default settings.
$client->indices()->create([
    'index' => $indexName,
    'body' => [
        'settings' => [
            'index' => [
                'number_of_shards' => 4
            ]
        ]
    ]
]);

$client->create([
    'index' => $indexName,
    'id' => 1,
    'body' => [
        'title' => 'Moneyball',
        'director' => 'Bennett Miller',
        'year' => 2011
    ]
]);

// Search for it
var_dump(
    $client->search([
        'index' => $indexName,
        'body' => [
            'size' => 5,
            'query' => [
                'multi_match' => [
                    'query' => 'miller',
                    'fields' => ['title^2', 'director']
                ]
            ]
        ]
    ])
);

// Delete a single document
$client->delete([
    'index' => $indexName,
    'id' => 1,
]);


// Delete index
$client->indices()->delete([
    'index' => $indexName
]);

?>
```
{％包括copy.html％


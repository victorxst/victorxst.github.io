---
layout: default
title: 语言客户端
nav_order: 1
has_children: false
nav_exclude: true
redirect_from:
  - /clients/
---

# OpenSearch语言客户端

OpenSearch为JavaScript，Python，Ruby，Java，Php，.net，Go and Rust提供了客户。

## OpenSearch客户端

OpenSearch为客户提供以下编程语言和平台的客户：

***Python**
  *[OpenSearch High-python客户端]({{site.url}}{{site.baseurl}}/clients/python-high-level/)
  *[OpenSearch低-python客户端]({{site.url}}{{site.baseurl}}/clients/python-low-level/)
  *[`opensearch-py-ml` 客户]({{site.url}}{{site.baseurl}}/clients/opensearch-py-ml/)
***爪哇**
  *[OpenSearch Java客户端]({{site.url}}{{site.baseurl}}/clients/java/)
***JavaScript**
  *[OpenSearch JavaScript（node.js）客户端]({{site.url}}{{site.baseurl}}/clients/javascript/index)
***去**
  *[OpenSearch Go客户端]({{site.url}}{{site.baseurl}}/clients/go/)
***红宝石**
  *[OpenSearch Ruby客户端]({{site.url}}{{site.baseurl}}/clients/ruby/)
***php**
  *[OpenSearch PHP客户端]({{site.url}}{{site.baseurl}}/clients/php/)
***。网**
  *[OpenSearch .NET客户端]({{site.url}}{{site.baseurl}}/clients/dot-net/)
***锈**
  *[OpenSearch Rust客户端]({{site.url}}{{site.baseurl}}/clients/rust/)

所有客户端都与任何版本的OpenSearch兼容。
{: .note}


## 旧客户

与Elasticsearch OSS 7.10.2 *合作的大多数客户都应该与OpenSearch一起使用，但是这些客户的最新版本可能包括人为折断兼容性的许可证或版本检查。此页面包括围绕哪些客户使用的建议，以与OpenSearch的最佳兼容性。

客户| 推荐版本
:--- | :---
[Elasticsearch Java Low-级别休息客户端](https://search.maven.org/artifact/org.elasticsearch.client/elasticsearch-rest-client/7.13.4/jar) | 7.13.4
[Elasticsearch Java High-级别休息客户端](https://search.maven.org/artifact/org.elasticsearch.client/elasticsearch-rest-high-level-client/7.13.4/jar) | 7.13.4
[Elasticsearch Python客户端](https://pypi.org/project/elasticsearch/7.13.4/) | 7.13.4
[Elasticsearch Node.js客户端](https://www.npmjs.com/package/@elastic/elasticsearch/v/7.13.0) | 7.13.0
[Elasticsearch Ruby客户端](https://rubygems.org/gems/elasticsearch/versions/7.13.0) | 7.13.0

如果您测试旧的客户端并确认其有效，请[提交公关](https://github.com/opensearch-project/documentation-website/pulls) 并将其添加到此表中。


{％ 评论 ％}
## Python 3测试代码

该代码索引一个文档，相当于`PUT /python-test-index1/_doc/1`。

```python
from elasticsearch import Elasticsearch

host = 'localhost'
port = 9200
# For testing only. Do not store credentials in code.
auth = ('admin', 'admin')

es = Elasticsearch(
  hosts = [{'host': host, 'port': port}],
  http_auth = auth,
  use_ssl = True,
  verify_certs = False
)

document = {
  "title": "Moneyball",
  "director": "Bennett Miller",
  "year": "2011"
}

response = es.index(index='python-test-index1', id='1', body=document, refresh=True)

print(response)
```


## Node.js测试代码

此代码等同于`GET /`。

```js
const { Client } = require('@elastic/elasticsearch')
const client = new Client({
  node: 'https://localhost:9200',
  auth: {
    // For testing only. Don't store credentials in code.
    username: 'admin',
    password: 'admin'
  },
  ssl: {
    // ca: fs.readFileSync('./cacert.pem'),
    rejectUnauthorized: false
  }
})

async function run () {
  const { body } = await client.info();
  console.log(body);
}

run().catch(console.log)
```
{％endcomment％}


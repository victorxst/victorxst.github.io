---
layout: default
title: Java高层REST客户端
nav_order: 20
---

# Java高层REST客户端

OpenSearch Java高-级别的REST客户端被弃用。支持将在OpenSearch版本3.0.0中删除。我们建议切换到[Java客户端]({{site.url}}{{site.baseurl}}/clients/java/) 反而。
{: .warning}

OpenSearch Java高-Level REST客户端可让您通过Java方法和数据结构而不是HTTP方法和JSON与OpenSearch群集和索引进行交互。

## 设置

开始使用OpenSearch Java高-升级休息客户端，确保您的项目具有以下依赖性`pom.xml` 文件：

```
<dependency>
  <groupId>org.opensearch.client</groupId>
  <artifactId>opensearch-rest-high-level-client</artifactId>
  <version>{{site.opensearch_version}}</version>
</dependency>
```

现在，您可以启动OpenSearch cluster。OpenSearch 1.x高-Level REST客户端可与OpenSearch的1.x版本一起使用。

## 安全

在Java应用程序中使用REST客户端之前，您必须配置应用程序的信托存储店以连接到安全插件。如果您正在使用自我-签名的证书或演示配置，您可以使用以下命令来创建自定义信托存储并添加根权限证书。

如果您使用来自受信任证书授权（CA）的证书，则无需配置信托店。

```bash
keytool -import <path-to-cert> -alias <alias-to-call-cert> -keystore <truststore-name>
```

现在，您可以将Java客户端指向TrustStore，并设置可以访问安全群集的基本身份验证凭据（请参阅下面的示例代码有关方法）。

如果您在配置安全性时会遇到问题，请参阅[常见问题]({{site.url}}{{site.baseurl}}/troubleshoot/index) 和[故障排除TLS]({{site.url}}{{site.baseurl}}/troubleshoot/tls)。

## 样本程序

此代码示例使用默认OpenSearch配置随附的基本凭据。如果您使用的是opensearch java高-使用您自己的OpenSearch集群将REST客户端放置，请确保更改代码以使用您自己的凭据。

```java
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.opensearch.action.admin.indices.delete.DeleteIndexRequest;
import org.opensearch.action.delete.DeleteRequest;
import org.opensearch.action.delete.DeleteResponse;
import org.opensearch.action.get.GetRequest;
import org.opensearch.action.get.GetResponse;
import org.opensearch.action.index.IndexRequest;
import org.opensearch.action.index.IndexResponse;
import org.opensearch.action.support.master.AcknowledgedResponse;
import org.opensearch.client.RequestOptions;
import org.opensearch.client.RestClient;
import org.opensearch.client.RestClientBuilder;
import org.opensearch.client.RestHighLevelClient;
import org.opensearch.client.indices.CreateIndexRequest;
import org.opensearch.client.indices.CreateIndexResponse;
import org.opensearch.common.settings.Settings;

import java.io.IOException;
import java.util.HashMap;

public class RESTClientSample {

  public static void main(String[] args) throws IOException {

    //Point to keystore with appropriate certificates for security.
    System.setProperty("javax.net.ssl.trustStore", "/full/path/to/keystore");
    System.setProperty("javax.net.ssl.trustStorePassword", "password-to-keystore");

    //Establish credentials to use basic authentication.
    //Only for demo purposes. Don't specify your credentials in code.
    final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();

    credentialsProvider.setCredentials(AuthScope.ANY,
      new UsernamePasswordCredentials("admin", "admin"));

    //Create a client.
    RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "https"))
      .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
        @Override
        public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
          return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
            }
          });
    RestHighLevelClient client = new RestHighLevelClient(builder);

    //Create a non-default index with custom settings and mappings.
    CreateIndexRequest createIndexRequest = new CreateIndexRequest("custom-index");

    createIndexRequest.settings(Settings.builder() //Specify in the settings how many shards you want in the index.
      .put("index.number_of_shards", 4)
      .put("index.number_of_replicas", 3)
      );
    //Create a set of maps for the index's mappings.
    HashMap<String, String> typeMapping = new HashMap<String,String>();
    typeMapping.put("type", "integer");
    HashMap<String, Object> ageMapping = new HashMap<String, Object>();
    ageMapping.put("age", typeMapping);
    HashMap<String, Object> mapping = new HashMap<String, Object>();
    mapping.put("properties", ageMapping);
    createIndexRequest.mapping(mapping);
    CreateIndexResponse createIndexResponse = client.indices().create(createIndexRequest, RequestOptions.DEFAULT);

    //Adding data to the index.
    IndexRequest request = new IndexRequest("custom-index"); //Add a document to the custom-index we created.
    request.id("1"); //Assign an ID to the document.

    HashMap<String, String> stringMapping = new HashMap<String, String>();
    stringMapping.put("message:", "Testing Java REST client");
    request.source(stringMapping); //Place your content into the index's source.
    IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);

    //Getting back the document
    GetRequest getRequest = new GetRequest("custom-index", "1");
    GetResponse response = client.get(getRequest, RequestOptions.DEFAULT);

    System.out.println(response.getSourceAsString());

    //Delete the document
    DeleteRequest deleteDocumentRequest = new DeleteRequest("custom-index", "1"); //Index name followed by the ID.
    DeleteResponse deleteResponse = client.delete(deleteDocumentRequest, RequestOptions.DEFAULT);

    //Delete the index
    DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("custom-index"); //Index name.
    AcknowledgedResponse deleteIndexResponse = client.indices().delete(deleteIndexRequest, RequestOptions.DEFAULT);

    client.close();
  }
}
```

## Elasticsearch Oss Java高-级别休息客户端

我们建议使用OpenSearch客户端连接到OpenSearch群集，但是如果您必须使用Elasticsearch OSS Java High-Level REST客户端，Elasticsearch OSS客户端的7.10.2版本还可以与OpenSearch的1.x版本一起使用。

### 迁移到OpenSearch Java高-级别休息客户端

从Elasticsearch OSS客户端迁移到OpenSearch High-级别的rest客户端就像将Maven依赖性更改为引用的依赖性一样简单[OpenSearch的依赖性](#setup)。

之后，更改所有参考`org.elasticsearch` 到`org.opensearch`，您准备开始向OpenSearch群集提交请求。


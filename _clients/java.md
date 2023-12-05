---
layout: default
title: Java客户端
nav_order: 30
---

# Java客户端

OpenSearch Java客户端使您可以通过Java方法和数据结构而不是HTTP方法和RAW JSON与OpenSearch群集进行交互。例如，您可以使用对象将请求提交到群集，以创建索引，将数据添加到文档中，或使用客户端的构建-在方法中。有关客户的完整API文档和其他示例，请参见[Javadoc](https://www.javadoc.io/doc/org.opensearch.client/opensearch-java/latest/index.html)。

该入门指南说明了如何连接到OpenSearch，索引文档和运行查询。有关客户端源代码，请参阅[OpenSearch-Java Repo](https://github.com/opensearch-project/opensearch-java)。

## 使用Apache HTTPClient 5 Transport安装客户端

要开始使用OpenSearch Java客户端，您需要提供运输。默认值`ApacheHttpClient5TransportBuilder` 运输带有Java客户。要使用默认传输的OpenSearch Java客户端，请将其添加到您的`pom.xml` 文件作为依赖性：

```xml
<dependency>
  <groupId>org.opensearch.client</groupId>
  <artifactId>opensearch-java</artifactId>
  <version>2.6.0</version>
</dependency>

<dependency>
  <groupId>org.apache.httpcomponents.client5</groupId>
  <artifactId>httpclient5</artifactId>
  <version>5.2.1</version>
</dependency>
```
{％include copy.html％}

如果您使用的是Gradle，请将以下依赖项添加到您的项目中：

```
dependencies {
  implementation 'org.opensearch.client:opensearch-java:2.6.0'
}
```
{％include copy.html％}

现在，您可以启动OpenSearch cluster。

## 使用RESTCLIENT运输安装客户端

另外，您可以使用`RestClient`-基于运输。在这种情况下，请确保您的项目中有以下依赖关系`pom.xml` 文件：

```xml
<dependency>
  <groupId>org.opensearch.client</groupId>
  <artifactId>opensearch-rest-client</artifactId>
  <version>{{site.opensearch_version}}</version>
</dependency>

<dependency>
  <groupId>org.opensearch.client</groupId>
  <artifactId>opensearch-java</artifactId>
  <version>2.6.0</version>
</dependency>
```
{％include copy.html％}

如果您使用的是Gradle，请将以下依赖项添加到您的项目中”

```
dependencies {
  implementation 'org.opensearch.client:opensearch-rest-client:{{site.opensearch_version}}'
  implementation 'org.opensearch.client:opensearch-java:2.6.0'
}
```
{％include copy.html％}

现在，您可以启动OpenSearch cluster。

## 安全

在Java应用程序中使用REST客户端之前，您必须配置应用程序的信托存储店以连接到安全插件。如果您正在使用自我-签名的证书或演示配置，您可以使用以下命令来创建自定义信托存储并添加根权限证书。

如果您使用来自受信任证书授权（CA）的证书，则无需配置信托店。

```bash
keytool -import <path-to-cert> -alias <alias-to-call-cert> -keystore <truststore-name>
```
{％include copy.html％}

现在，您可以将Java客户端指向TrustStore，并设置可以访问安全群集的基本身份验证凭据（请参阅下面的示例代码有关方法）。

如果您在配置安全性时会遇到问题，请参阅[常见问题]({{site.url}}{{site.baseurl}}/troubleshoot/index) 和[故障排除TLS]({{site.url}}{{site.baseurl}}/troubleshoot/tls)。

## 样本数据

本节使用称为`IndexData`，这是一个简单的Java类，它存储基本的数据和方法。对于自己的OpenSearch集群，您可能会发现您需要一个更强大的类来存储数据。

### indexdata类

```java
static class IndexData {
  private String firstName;
  private String lastName;

  public IndexData(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  public String getFirstName() {
    return firstName;
  }

  public void setFirstName(String firstName) {
    this.firstName = firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public void setLastName(String lastName) {
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return String.format("IndexData{first name='%s', last name='%s'}", firstName, lastName);
  }
}
```
{％include copy.html％}

## 使用Apache HTTPClient 5 Transpers启用SSL和TLS初始化客户端

此代码示例使用默认OpenSearch配置随附的基本凭据。如果您使用自己的OpenSearch集群使用Java客户端，请确保更改代码，以便使用您自己的凭据。

以下示例代码启用了SSL和TLS的客户端：


```java
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLEngine;

import org.apache.hc.client5.http.auth.AuthScope;
import org.apache.hc.client5.http.auth.UsernamePasswordCredentials;
import org.apache.hc.client5.http.impl.auth.BasicCredentialsProvider;
import org.apache.hc.client5.http.impl.nio.PoolingAsyncClientConnectionManager;
import org.apache.hc.client5.http.impl.nio.PoolingAsyncClientConnectionManagerBuilder;
import org.apache.hc.client5.http.ssl.ClientTlsStrategyBuilder;
import org.apache.hc.core5.function.Factory;
import org.apache.hc.core5.http.HttpHost;
import org.apache.hc.core5.http.nio.ssl.TlsStrategy;
import org.apache.hc.core5.reactor.ssl.TlsDetails;
import org.apache.hc.core5.ssl.SSLContextBuilder;
import org.opensearch.client.opensearch.OpenSearchClient;
import org.opensearch.client.transport.OpenSearchTransport;
import org.opensearch.client.transport.httpclient5.ApacheHttpClient5TransportBuilder;

public class OpenSearchClientExample {
  public static void main(String[] args) throws Exception {
    System.setProperty("javax.net.ssl.trustStore", "/full/path/to/keystore");
    System.setProperty("javax.net.ssl.trustStorePassword", "password-to-keystore");

    final HttpHost host = new HttpHost("https", "localhost", 9200);
    final BasicCredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    // Only for demo purposes. Don't specify your credentials in code.
    credentialsProvider.setCredentials(new AuthScope(host), new UsernamePasswordCredentials("admin", "admin".toCharArray()));

    final SSLContext sslcontext = SSLContextBuilder
      .create()
      .loadTrustMaterial(null, (chains, authType) -> true)
      .build();

    final ApacheHttpClient5TransportBuilder builder = ApacheHttpClient5TransportBuilder.builder(host);
    builder.setHttpClientConfigCallback(httpClientBuilder -> {
      final TlsStrategy tlsStrategy = ClientTlsStrategyBuilder.create()
        .setSslContext(SSLContextBuilder.create().build())
        // See https://issues.apache.org/jira/browse/HTTPCLIENT-2219
        .setTlsDetailsFactory(new Factory<SSLEngine, TlsDetails>() {
          @Override
          public TlsDetails create(final SSLEngine sslEngine) {
            return new TlsDetails(sslEngine.getSession(), sslEngine.getApplicationProtocol());
          }
        })
        .build();

      final PoolingAsyncClientConnectionManager connectionManager = PoolingAsyncClientConnectionManagerBuilder
        .create()
        .setTlsStrategy(tlsStrategy)
        .build();

      return httpClientBuilder
        .setDefaultCredentialsProvider(credentialsProvider)
        .setConnectionManager(connectionManager);
    });

    final OpenSearchTransport transport = ApacheHttpClient5TransportBuilder.builder(host).build();
    OpenSearchClient client = new OpenSearchClient(transport);
  }
}

```

## 使用RESTCLIENT TRANSPORM启用SSL和TLS初始化客户端

此代码示例使用默认OpenSearch配置随附的基本凭据。如果您使用自己的OpenSearch集群使用Java客户端，请确保更改代码，以便使用您自己的凭据。

以下示例代码启用了SSL和TLS的客户端：

```java
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.opensearch.client.RestClient;
import org.opensearch.client.RestClientBuilder;
import org.opensearch.client.json.jackson.JacksonJsonpMapper;
import org.opensearch.client.opensearch.OpenSearchClient;
import org.opensearch.client.transport.OpenSearchTransport;
import org.opensearch.client.transport.rest_client.RestClientTransport;

public class OpenSearchClientExample {
  public static void main(String[] args) throws Exception {
    System.setProperty("javax.net.ssl.trustStore", "/full/path/to/keystore");
    System.setProperty("javax.net.ssl.trustStorePassword", "password-to-keystore");

    final HttpHost host = new HttpHost("https", 9200, "localhost");
    final BasicCredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    //Only for demo purposes. Don't specify your credentials in code.
    credentialsProvider.setCredentials(new AuthScope(host), new UsernamePasswordCredentials("admin", "admin"));

    //Initialize the client with SSL and TLS enabled
    final RestClient restClient = RestClient.builder(host).
      setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
        @Override
        public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
        }
      }).build();

    final OpenSearchTransport transport = new RestClientTransport(restClient, new JacksonJsonpMapper());
    final OpenSearchClient client = new OpenSearchClient(transport);
  }
}
```
{％include copy.html％}

## 连接到Amazon OpenSearch服务

下面的示例说明了连接到Amazon OpenSearch服务：

```java
SdkHttpClient httpClient = ApacheHttpClient.builder().build();

OpenSearchClient client = new OpenSearchClient(
    new AwsSdk2Transport(
        httpClient,
        "search-...us-west-2.es.amazonaws.com", // OpenSearch endpoint, without https://
        "es"
        Region.US_WEST_2, // signing service region
        AwsSdk2TransportOptions.builder().build()
    )
);

InfoResponse info = client.info();
System.out.println(info.version().distribution() + ": " + info.version().number());

httpClient.close();
```
{％include copy.html％}

## 连接到Amazon OpenSearch无服务器

以下示例说明了连接到Amazon OpenSearch无服务器服务：

```java
SdkHttpClient httpClient = ApacheHttpClient.builder().build();

OpenSearchClient client = new OpenSearchClient(
    new AwsSdk2Transport(
        httpClient,
        "search-...us-west-2.aoss.amazonaws.com", // OpenSearch endpoint, without https://
        "aoss"
        Region.US_WEST_2, // signing service region
        AwsSdk2TransportOptions.builder().build()
    )
);

InfoResponse info = client.info();
System.out.println(info.version().distribution() + ": " + info.version().number());

httpClient.close();
```
{％include copy.html％}


## 创建索引

您可以使用非-使用以下代码的默认设置：

```java
String index = "sample-index";
CreateIndexRequest createIndexRequest = new CreateIndexRequest.Builder().index(index).build();
client.indices().create(createIndexRequest);

IndexSettings indexSettings = new IndexSettings.Builder().autoExpandReplicas("0-all").build();
PutIndicesSettingsRequest putIndicesSettingsRequest = new PutIndicesSettingsRequest.Builder().index(index).value(indexSettings).build();
client.indices().putSettings(putIndicesSettingsRequest);
```
{％include copy.html％}

## 索引数据

您可以使用以下代码将数据索引到OpenSearch：

```java
IndexData indexData = new IndexData("first_name", "Bruce");
IndexRequest<IndexData> indexRequest = new IndexRequest.Builder<IndexData>().index(index).id("1").document(indexData).build();
client.index(indexRequest);
```
{％include copy.html％}

## 搜索文档

您可以使用以下代码搜索文档：

```java
SearchResponse<IndexData> searchResponse = client.search(s -> s.index(index), IndexData.class);
for (int i = 0; i< searchResponse.hits().hits().size(); i++) {
  System.out.println(searchResponse.hits().hits().get(i).source());
}
```
{％include copy.html％}

## 删除文档

以下示例代码删除ID为1的文档：

```java
client.delete(b -> b.index(index).id("1"));
```
{％include copy.html％}

### 删除索引

以下示例代码删除索引：

```java
DeleteIndexRequest deleteIndexRequest = new DeleteRequest.Builder().index(index).build();
DeleteIndexResponse deleteIndexResponse = client.indices().delete(deleteIndexRequest);
```
{％include copy.html％}

## 样本程序

以下示例程序创建了一个客户端，添加了一个非索引-默认设置，插入文档，搜索文档，删除文档，然后删除索引：

```java
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.opensearch.client.RestClient;
import org.opensearch.client.RestClientBuilder;
import org.opensearch.client.base.RestClientTransport;
import org.opensearch.client.base.Transport;
import org.opensearch.client.json.jackson.JacksonJsonpMapper;
import org.opensearch.client.opensearch.OpenSearchClient;
import org.opensearch.client.opensearch._global.IndexRequest;
import org.opensearch.client.opensearch._global.IndexResponse;
import org.opensearch.client.opensearch._global.SearchResponse;
import org.opensearch.client.opensearch.indices.*;
import org.opensearch.client.opensearch.indices.put_settings.IndexSettingsBody;

import java.io.IOException;

public class OpenSearchClientExample {
  public static void main(String[] args) {
    RestClient restClient = null;
    try{
      System.setProperty("javax.net.ssl.trustStore", "/full/path/to/keystore");
      System.setProperty("javax.net.ssl.trustStorePassword", "password-to-keystore");

      //Only for demo purposes. Don't specify your credentials in code.
      final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
      credentialsProvider.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("admin", "admin"));

      //Initialize the client with SSL and TLS enabled
      restClient = RestClient.builder(new HttpHost("localhost", 9200, "https")).
        setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
          @Override
          public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
          return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
          }
        }).build();
      Transport transport = new RestClientTransport(restClient, new JacksonJsonpMapper());
      OpenSearchClient client = new OpenSearchClient(transport);

      //Create the index
      String index = "sample-index";
      CreateIndexRequest createIndexRequest = new CreateIndexRequest.Builder().index(index).build();
      client.indices().create(createIndexRequest);

      //Add some settings to the index
      IndexSettings indexSettings = new IndexSettings.Builder().autoExpandReplicas("0-all").build();
      IndexSettingsBody settingsBody = new IndexSettingsBody.Builder().settings(indexSettings).build();
      PutSettingsRequest putSettingsRequest = new PutSettingsRequest.Builder().index(index).value(settingsBody).build();
      client.indices().putSettings(putSettingsRequest);

      //Index some data
      IndexData indexData = new IndexData("first_name", "Bruce");
      IndexRequest<IndexData> indexRequest = new IndexRequest.Builder<IndexData>().index(index).id("1").document(indexData).build();
      client.index(indexRequest);

      //Search for the document
      SearchResponse<IndexData> searchResponse = client.search(s -> s.index(index), IndexData.class);
      for (int i = 0; i< searchResponse.hits().hits().size(); i++) {
        System.out.println(searchResponse.hits().hits().get(i).source());
      }

      //Delete the document
      client.delete(b -> b.index(index).id("1"));

      // Delete the index
      DeleteIndexRequest deleteIndexRequest = new DeleteRequest.Builder().index(index).build();
      DeleteIndexResponse deleteIndexResponse = client.indices().delete(deleteIndexRequest);
    } catch (IOException e){
      System.out.println(e.toString());
    } finally {
      try {
        if (restClient != null) {
          restClient.close();
        }
      } catch (IOException e) {
        System.out.println(e.toString());
      }
    }
  }
}
```
{％include copy.html％}


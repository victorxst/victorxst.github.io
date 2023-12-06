---
layout: default
title: 会话搜索
has_children: false
nav_order: 200
---

这是一个实验特征，不建议在生产环境中使用。有关功能进度或要留下反馈的更新，请参阅关联[Github问题](https://forum.opensearch.org/t/feedback-conversational-search-and-retrieval-augmented-generation-using-search-pipeline-experimental-release/16073)。
{: .warning}

# 会话搜索

会话搜索是一种实验机学习（ML）功能，可实现新的搜索接口。尽管传统的文档搜索使您可以提出问题并收到可能包含该问题答案的文档列表，但会话搜索使用大型语言模型（LLMS）读取顶级n个文档并将这些文档综合为专业"answer" 问你的问题。

当前，对话搜索使用两个系统来合成文档：

- [对话记忆](#conversation-memory)
- [检索增强发电（RAG）管道](#rag-pipeline)

## 对话记忆

对话记忆包括一个简单的碎屑-就像API包括两个资源：**对话** 和**互动**。对话是由互动组成的。互动代表一对消息：人类输入和人工智能（AI）响应。在创建对话之前，您无法创建任何交互。

为了使使用对话内存的构建和调试应用程序更容易，`conversation-meta` 和`conversation-interactions` 存储在两个系统索引中。

### `conversation-meta` 指数

在里面`conversation-meta` 索引，您可以自定义`name` 字段使最终用户更容易知道如何继续与AI进行对话，如以下模式所示：

```jsx
.plugins-ml-conversation-meta
{
    "_meta": {
        "schema_version": 1
    },
    "properties": {
        "name": {"type": "keyword"},
        "create_time": {"type": "date", "format": "strict_date_time||epoch_millis"},
        "user": {"type": "keyword"}
    }
}
```

### `conversation-interactions` 指数

在里面`conversation-interactions` 索引，以下所有字段均由用户或AI应用程序设置。每个字段以字符串输入。

| 场地| 描述|
| :--- | :--- |
| `input` | 构成互动基础的问题。|
| `prompt_template` | 用作此交互的框架的提示模板。|
| `response` | AI对提示的响应。|
| `origin` | 生成响应的AI或其他系统的名称。|
| `additional_info` | 发送给的任何其他信息"origin" 在提示中。|

这`conversation-interactions` 索引创建了一个干净的交互抽象，并使索引可以轻松地重建发送到LLM的确切提示，从而实现了可靠的调试和解释性，如以下模式：

```jsx
.plugins-ml-conversation-interactions
{
    "_meta": {
        "schema_version": 1
    },
    "properties": {
        "conversation_id": {"type": "keyword"},
        "create_time": {"type": "date", "format": "strict_date_time||epoch_millis"},
        "input": {"type": "text"},
        "prompt_template": {"type": "text"},
        "response": {"type": "text"},
        "origin": {"type": "keyword"},
        "additional_info": {"type": "text"}
    }
}
```

## 处理对话和互动

启用安全插件后，ML Commons中的所有对话都存在于"private" 安全模式。只有创建对话的用户才能与该对话进行交互。集群上没有用户可以看到另一个用户的对话。
{: .note}

要开始使用对话内存，请启用以下群集设置：

```json
PUT /_cluster/settings
{
  "persistent": {
    "plugins.ml_commons.memory_feature_enabled": true
  }
}
```
{% include copy-curl.html %}

启用对话内存后，您可以使用内存API创建对话。

要使对话易于识别，请使用可选的`name` 内存API中的字段，如下面的示例所示。这将是您命名对话的唯一机会。



```json
POST /_plugins/_ml/memory/conversation
{
  "name": Example conversation
}
```
{% include copy-curl.html %}

内存API使用对话ID响应，如以下示例响应所示：

```json
{ "conversation_id": "4of2c9nhoIuhcr" }
```

您将使用`conversation_id` 在对话中创建互动。要创建互动，请输入`conversation_id` 进入内存API路径。然后自定义[字段](#conversation-interactions-index) 在请求主体中，如下所示：

```json
POST /_plugins/_ml/memory/conversation/4of2c9nhoIuhcr
{
  "input": "How do I make an interaction?",
  "prompt_template": "Hello OpenAI, can you answer this question? \
											Here's some extra info that may help. \
											[INFO] \n [QUESTION]",
  "response": "Hello, this is OpenAI. Here is the answer to your question.",
  "origin": "MyFirstOpenAIWrapper",
  "additional_info": "Additional text related to the answer \
											A JSON or other semi-structured response"
}
```
{% include copy-curl.html %}

然后，内存API以相互作用ID响应，如以下示例响应所示：

```json
{ "interaction_id": "948vh_PoiY2hrnpo" }
```

### 进行对话

您可以使用以下内存API操作获取对话列表：

```json
GET /_plugins/_ml/memory/conversation?max_results=3&next_token=0
```
{% include copy-curl.html %}

使用以下路径参数自定义结果。

范围| 数据类型| 描述
:--- | :--- | :---
`max_results` | 整数| 响应返回的最大结果数。默认为`10`。
`next_token` | 整数| 表示将要检索的对话顺序位置。例如，如果对话A，B和C存在，`next_token=1` 将返回对话b和C.默认值为`0`。

内存API通过最近的对话做出响应，如`create_time` 以下示例响应的字段：

```json
{
  "conversations": [
    {
      "conversation_id": "0y4hto_in1",
      "name": "",
      "create_time": "2023-4-23 10:25.324662"
	  }, ... (2 more since we specified max_results=3)
	],
  "next_token": 3
}
```


如果对话少于设置的数字`max_results`，响应仅返回存在的对话数量。最后，`next_token` 提供对话的排序列表的有序位置。当随后的获取对话呼叫之间添加对话时，结果将在结果中重复：

```plaintext
GetConversations               -> [BCD]EFGH
CreateConversation             -> ABCDEFGH
GetConversations(next_token=3) -> ABC[DEF]GH
```

### 获得互动

要查看对话中的互动列表，请输入`conversation_id` 在API请求结束时，如下示例所示。您可以使用`max_results` 和`next_token` 对响应进行排序：

```json
GET /_plugins/_ml/memory/conversation/4of2c9nhoIuhcr
```
{% include copy-curl.html %}

内存API返回以下交互信息：

```json
{
  "interactions": [
    {
      "interaction_id": "342968y2u4-0",
      "conversation_id": "0y4hto_in1",
      "create_time": "2023-4-23 10:25.324662",
      "input": "How do I make an interaction?",
      "prompt_template": "Hello OpenAI, can you answer this question? \
											Here's some extra info that may help. \
											[INFO] \n [QUESTION]",
      "response": "Hello, this is OpenAI. Here is the answer to your question.",
      "origin": "MyFirstOpenAIWrapper",
      "additional_info": "Additional text related to the answer \
											A JSON or other semi-structured response"
	  }, ... (9 more since max_results defaults to 10)
	],
  "next_token": 10
}
```

### 删除对话

要删除对话，请使用`DELETE` 操作，如以下示例所示：

```json
DELETE /_plugins/_ml/memory/conversation/4of2c9nhoIuhcr
```
{% include copy-curl.html %}

内存API响应以下响应：

```json
{ "success": true }
```

## 破布管道

RAG是一种从索引中检索文档的技术，通过SEQ2SEQ模型（例如LLM）将其传递，然后在上下文中使用动态检索的数据来补充静态LLM信息。

从OpenSearch 2.11开始，仅使用OpenAI型号和Amazon Bedrock上的Anthropic Claude模型对RAG技术进行了测试。
{: .warning}

### 启用抹布

使用以下群集设置启用RAG管道功能：

```json
PUT /_cluster/settings
{
  "persistent": {"plugins.ml_commons.rag_pipeline_feature_enabled": "true"}
}
```
{% include copy-curl.html %}

### 连接模型

RAG需要LLM才能运行。我们建议使用[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。

使用以下步骤使用OpenAI GPT 3.5模型设置HTTP连接器：

1. 使用连接器API创建HTTP连接器：

    ```json
    POST /_plugins/_ml/connectors/_create
    {
        "name": "OpenAI Chat Connector",
        "description": "The connector to public OpenAI model service for GPT 3.5",
        "version": 2,
        "protocol": "http",
        "parameters": {
            "endpoint": "api.openai.com",
            "model": "gpt-3.5-turbo",
      "temperature": 0
        },
        "credential": {
            "openAI_key": "<your OpenAI key>"
        },
        "actions": [
            {
                "action_type": "predict",
                "method": "POST",
                "url": "https://${parameters.endpoint}/v1/chat/completions",
                "headers": {
                    "Authorization": "Bearer ${credential.openAI_key}"
                },
                "request_body": "{ \"model\": \"${parameters.model}\", \"messages\": ${parameters.messages}, \"temperature\": ${parameters.temperature} }"
            }
        ]
    }
    ```
    {% include copy-curl.html %}

1. Create a new model group for the connected model. You'll use the `model_group_id` returned by the Register API to register the model:

    ```json
    POST /_plugins/_ml/model_groups/_register
    {
      "name": "public_model_group", 
      "description": "This is a public model group"
    }
    ```
    {% include copy-curl.html %}

1. Register and deploy the model using the `connector_id` from the Connector API response in Step 1 and the `model_group_id` 在步骤2中返回：

    ```json
    POST /_plugins/_ml/models/_register
    {
      "name": "openAI-gpt-3.5-turbo",
      "function_name": "remote",
      "model_group_id": "fp-hSYoBu0R6vVqGMnM1",
      "description": "test model",
      "connector_id": "f5-iSYoBu0R6vVqGI3PA"
    }
    ``` 
    {% include copy-curl.html %}

1. 在注册模型后，使用`task_id` 在注册响应中返回以获取`model_id`。您将使用`model_id` 将模型部署到OpenSearch：

    ```json
    GET /_plugins/_ml/tasks/<task_id>
    ```
    {% include copy-curl.html %}

1. Using the `model_id` from step 4, deploy the model:

    ```json
    POST /_plugins/_ml/models/<model_id>/_deploy
    ```
    {% include copy-curl.html %}

### Setting up the pipeline

Next, you'll create a search pipeline for the connector model. Use the following Search API request to create a pipeline: 

```json
PUT /_search/pipeline/<pipeline_name>
{
  "response_processors": [
    {
      "retrieval_augmented_generation": {
        "tag": "openai_pipeline_demo",
        "description": "Demo pipeline Using OpenAI Connector",
        "model_id": "<model_id>",
        "context_field_list": ["text"],
        "system_prompt": "You are a helpful assistant",
        "user_instructions": "Generate a concise and informative answer in less than 100 words for the given question"
      }
    }
  ]
}
```
{% include copy-curl.html %}

### Context field list

`context_field_list` is the list of fields in document sources that the pipeline uses as context for the RAG. For example, when `context_field_list` parses through the following document, the pipeline sends the `文本` 从响应对OpenAI模型的范围：

```json
{
  "_index": "qa_demo",
  "_id": "SimKcIoBOVKVCYpk1IL-",
  "_source": {
    "title": "Abraham Lincoln 2",
    "text": "Abraham Lincoln was born on February 12, 1809, the second child of Thomas Lincoln and Nancy Hanks Lincoln, in a log cabin on Sinking Spring Farm near Hodgenville, Kentucky.[2] He was a descendant of Samuel Lincoln, an Englishman who migrated from Hingham, Norfolk, to its namesake, Hingham, Massachusetts, in 1638. The family then migrated west, passing through New Jersey, Pennsylvania, and Virginia.[3] Lincoln was also a descendant of the Harrison family of Virginia; his paternal grandfather and namesake, Captain Abraham Lincoln and wife Bathsheba (née Herring) moved the family from Virginia to Jefferson County, Kentucky.[b] The captain was killed in an Indian raid in 1786.[5] His children, including eight-year-old Thomas, Abraham's father, witnessed the attack.[6][c] Thomas then worked at odd jobs in Kentucky and Tennessee before the family settled in Hardin County, Kentucky, in the early 1800s.[6]\n"
  }
}
```

您可以自定义`context_field_list` 在您的RAG管道中，将文档中存在的任何字段发送到LLM。

### RAG参数选项

在设置RAG管道下方时使用以下选项`retrieval_augmented_generation` 争论。

范围| 必需的| 描述
:--- | :--- | :---
`tag` | 不| 标签以帮助识别管道。
`description` | 是的| 管道的描述。
`model_id` | 是的| 管道中使用的模型的ID。
`context_field_list` | 是的| 管道来源中的字段列表将管道用作抹布的上下文。有关更多信息，请参阅[上下文字段列表](#context-field-list)。
`system_prompt` | 不| 消息发送给LLM的消息`system` 角色。这是用户在LLM接收交互时看到的消息。
`user_instructions` | 不| LLM发送的另一条消息`user` 角色。此参数允许进一步自定义用户与LLM交互时收到的内容。

### 使用管道

使用管道类似于提交[搜索查询]({{site.url}}{{site.baseurl}}/api-reference/search/#example) 进行搜索，如以下示例所示：

```json
GET /<index_name>/_search?search_pipeline=<pipeline_name>
{
	"query" : {...},
	"ext": {
		"generative_qa_parameters": {
			"llm_model": "gpt-3.5-turbo",
			"llm_question": "Was Abraham Lincoln a good politician",
			"conversation_id": "_ikaSooBHvd8_FqDUOjZ",
                         "context_size": 5,
                         "interaction_size": 5,
                         "timeout": 15
		}
	}
}
```
{% include copy-curl.html %}

抹布搜索查询使用以下请求对象`generative_qa_parameters` 选项。

范围| 必需的| 描述
:--- | :--- | :---
`llm_question` | 是的| LLM必须回答的问题。
`llm_model` | 不| 在您要使用其他模型的情况下（例如，GPT 4而不是GPT 3.5），在连接中设置了原始模型。如果未在管道创建期间设置默认模型，则需要此选项。
`conversation_id` | 不| 通过将对话内存添加到LLM的搜索查询上下文中，将对话存储器集成到您的破布管道中。
`context_size` | 不| 发送到LLM的搜索结果数量。为了满足令牌尺寸限制，通常需要这这可能会因模型而异。或者，您可以使用`size` 搜索API中的参数以控制发送到LLM的信息量。
`interaction_size` | 不| 发送到LLM的互动数量。与搜索结果的数量相似，这会影响LLM看到的令牌总数。当未设置时，管道使用默认交互的大小`10`。
`timeout` | 不| 管道使用连接器响应远程模型的秒数。默认为`30`。

如果您的LLM包含集合令牌限制，请设置`size` OpenSearch查询中的字段以限制搜索响应中使用的文档数量。否则，RAG管道将将搜索结果中的每个文档发送到LLM。

## 下一步

- 要了解有关连接到外部平台上的模型的更多信息，请参见[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。
- 要了解有关在OpenSearch集群中使用自定义模型的更多信息，请参见[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。



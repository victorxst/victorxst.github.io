---
layout: default
title: 地图统计API
nav_order: 20
grand_parent: 构建数据可视化
parent: 使用坐标和区域图
has_children: false
---

# 地图统计API
引入2.7
{: .label .label-purple }

当您创建并保存一个[地图]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/) 在OpenSearch仪表板中，该地图变成了类型的保存对象`map`。地图统计信息API在OpenSearch仪表板中提供了有关此类保存对象的信息。

#### 示例请求

您可以通过以下格式提供其URL地址来访问MAPS统计信息：

```
<opensearch-dashboards-endpoint-address>/api/maps-dashboards/stats
```

如果OpenSearch Configuration文件中指定了OpenSearch仪表板端点地址，则可能包含一个端口号。特定的URL格式取决于OpenSearch部署的类型及其托管的网络环境的类型。
{: .note}

您可以通过两种方式查询端点：
  
  - 通过访问端点地址（例如，`http://localhost:5601/api/maps-dashboards/stats`）在浏览器中

  - 通过使用`curl` 终端中的命令：
    ```bash
    curl -X GET http://localhost:5601/api/maps-dashboards/stats
    ```
    {% include copy.html %}

#### 示例响应

以下是前面请求的响应：

```json
{
   "maps_total":4,  
   "layers_filters_total":4, 
   "layers_total":{ 
      "opensearch_vector_tile_map":2, 
      "documents":7, 
      "wms":1, 
      "tms":2 
   },
   "maps_list":[
      {
         "id":"88a24e6c-0216-4f76-8bc7-c8db6c8705da", 
         "layers_filters_total":4,
         "layers_total":{
            "opensearch_vector_tile_map":1,
            "documents":3,
            "wms":0,
            "tms":0
         }
      },
      {
         "id":"4ce3fe50-d309-11ed-a958-770756e00bcd",
         "layers_filters_total":0,
         "layers_total":{
            "opensearch_vector_tile_map":0,
            "documents":2,
            "wms":0,
            "tms":1
         }
      },
      {
         "id":"af5d3b90-d30a-11ed-a605-f7ad7bc98642",
         "layers_filters_total":0,
         "layers_total":{
            "opensearch_vector_tile_map":1,
            "documents":1,
            "wms":0,
            "tms":1
         }
      },
      {
         "id":"5ca1ec10-d30b-11ed-a042-93d8ff0f09ee",
         "layers_filters_total":0,
         "layers_total":{
            "opensearch_vector_tile_map":0,
            "documents":1,
            "wms":1,
            "tms":0
         }
      }
   ]
}
```

## 响应字段

响应包含以下层类型的统计信息：

- 基础胶质：默认的OpenSearch地图或自定义基本层映射。

- WMS层：自定义WMS基层图。

- TMS层：自定义TMS基层图。

- 文档层：地图的数据层。

有关图层类型的更多信息，请参见[添加图层]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/#adding-layers)。

下表列出了所有响应字段。

| 场地| 数据类型| 描述|
| :--- | :--- | :--- | 
| `maps_total` | 整数| 用地图插件注册为保存对象的地图总数。|
| `layers_filters_total` | 整数| 所有地图中所有层的过滤器总数。这包括[层-级过滤器]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/#filtering-data-at-the-layer-level) 但不包括全球过滤器[形状过滤器]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/#drawing-shapes-to-filter-data)。|
| `layers_total` | 目的| 所有地图中所有图层的总统计信息。|
| `layers_total.opensearch_vector_tile_map` | 整数| 所有地图中的OpenSearch基础胶质总数。|
| `layers_total.documents` | 整数| 所有地图中的文档层总数。|
| `layers_total.wms` | 整数| 所有地图中的WMS层总数。|
| `layers_total.tms` | 整数| 所有地图中TMS层的总数。|
| `maps_list` | 大批| 保存在OpenSearch仪表板中的所有地图的列表。|

每个地图`map_list` 包含以下字段。

| 场地| 数据类型| 描述|
| :--- | :--- | :--- | 
| `id` | 细绳| 地图保存的对象ID。|
| `layers_filters_total` | 整数| 地图中所有层的过滤器总数。这包括[层-级过滤器]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/#filtering-data-at-the-layer-level) 但不包括全球过滤器[形状过滤器]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/#drawing-shapes-to-filter-data) 。|
| `layers_total` | 目的| 地图中所有层的总统计信息。|
| `layers_total.opensearch_vector_tile_map` | 整数| 地图中的OpenSearch基本封面总数。|
| `layers_total.documents` | 整数| 地图中文档层的总数。|
| `layers_total.wms` | 整数| 地图中的WMS层的总数。|
| `layers_total.tms` | 整数| 地图中TMS的总数。|

保存的对象ID可帮助您导航到特定地图，因为ID是地图URL的最后部分。例如，在OpenSearch游乐场，地址`[Flights] Flights Status on Maps Destination Location` 地图是`https://playground.opensearch.org/app/maps-dashboards/88a24e6c-0216-4f76-8bc7-c8db6c8705da`， 在哪里`88a24e6c-0216-4f76-8bc7-c8db6c8705da` 是此地图的保存对象ID。
{: .tip}


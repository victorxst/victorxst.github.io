---
layout: default
title: 配置网络地图服务（WMS）
grand_parent: 构建数据可视化
parent: 使用坐标和区域图
nav_order: 30
redirect_from:
  - /dashboards/maptiles/
---

{％- 评论-％}这`/docs/opensearch-dashboards/maptiles/` 重定向专门支持OpenSearch仪表板1.0.0中的UI链接。{％- 终点-％}

# 配置网络地图服务（WMS）

开放的地理空间联盟（OGC）网络地图服务（WMS）规范是一种国际规范，用于在网络上要求动态地图。OpenSearch仪表板包括默认地图图块。对于专用地图，您可以按照以下步骤在OpenSearch仪表板上配置WMS：

1. 登录到OpenSearch仪表板`https://<host>:<port>`。例如，您可以通过连接到OpenSearch仪表板连接到OpenSearch仪表板[https：// localhost：5601](https://localhost:5601)。默认用户名和密码是`admin`。
2. 选择**管理** >**高级设置**。
3. 定位`visualization:tileMap:WMSdefaults`。
4. 改变`enabled` 到`true` 并添加有效的WMS服务器的URL，如以下示例所示：

   ```JSON
   {
     "enabled"： 真的，
     "url"："<wms-map-server-url>"，
     "options"：{
       "format"："image/png"，
       "transparent"： 真的
     }
   }
   ```

网络地图服务可能有许可费或限制，您有责任遵守任何此类费用或限制。
{: .note }


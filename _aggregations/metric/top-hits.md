---
layout: default
title: Top hits
parent: 公制聚合
grand_parent: 聚合
nav_order: 130
redirect_from:
  - /query-dsl/aggregations/metric/top-hits/
---

# 最高命中聚合

这`top_hits` 公制是多型-值度量集合根据正在汇总的字段的相关性分数对匹配文档进行排名。

您可以指定以下选项：

- `from`：命中的首发位置。
- `size`：返回的最大命中尺寸。默认值为3。
- `sort`：如何分类匹配的命中。默认情况下，命中按聚合查询的相关得分进行排序。

以下示例返回您的电子商务数据中的前5个产品：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "top_hits_products": {
      "top_hits": {
        "size": 5
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
...
"aggregations" : {
  "top_hits_products" : {
    "hits" : {
      "total" : {
        "value" : 4675,
        "relation" : "eq"
      },
      "max_score" : 1.0,
      "hits" : [
        {
          "_index" : "opensearch_dashboards_sample_data_ecommerce",
          "_type" : "_doc",
          "_id" : "glMlwXcBQVLeQPrkHPtI",
          "_score" : 1.0,
          "_source" : {
            "category" : [
              "Women's Accessories",
              "Women's Clothing"
            ],
            "currency" : "EUR",
            "customer_first_name" : "rania",
            "customer_full_name" : "rania Evans",
            "customer_gender" : "FEMALE",
            "customer_id" : 24,
            "customer_last_name" : "Evans",
            "customer_phone" : "",
            "day_of_week" : "Sunday",
            "day_of_week_i" : 6,
            "email" : "rania@evans-family.zzz",
            "manufacturer" : [
              "Tigress Enterprises"
            ],
            "order_date" : "2021-02-28T14:16:48+00:00",
            "order_id" : 583581,
            "products" : [
              {
                "base_price" : 10.99,
                "discount_percentage" : 0,
                "quantity" : 1,
                "manufacturer" : "Tigress Enterprises",
                "tax_amount" : 0,
                "product_id" : 19024,
                "category" : "Women's Accessories",
                "sku" : "ZO0082400824",
                "taxless_price" : 10.99,
                "unit_discount_amount" : 0,
                "min_price" : 5.17,
                "_id" : "sold_product_583581_19024",
                "discount_amount" : 0,
                "created_on" : "2016-12-25T14:16:48+00:00",
                "product_name" : "Snood - white/grey/peach",
                "price" : 10.99,
                "taxful_price" : 10.99,
                "base_unit_price" : 10.99
              },
              {
                "base_price" : 32.99,
                "discount_percentage" : 0,
                "quantity" : 1,
                "manufacturer" : "Tigress Enterprises",
                "tax_amount" : 0,
                "product_id" : 19260,
                "category" : "Women's Clothing",
                "sku" : "ZO0071900719",
                "taxless_price" : 32.99,
                "unit_discount_amount" : 0,
                "min_price" : 17.15,
                "_id" : "sold_product_583581_19260",
                "discount_amount" : 0,
                "created_on" : "2016-12-25T14:16:48+00:00",
                "product_name" : "Cardigan - grey",
                "price" : 32.99,
                "taxful_price" : 32.99,
                "base_unit_price" : 32.99
              }
            ],
            "sku" : [
              "ZO0082400824",
              "ZO0071900719"
            ],
            "taxful_total_price" : 43.98,
            "taxless_total_price" : 43.98,
            "total_quantity" : 2,
            "total_unique_products" : 2,
            "type" : "order",
            "user" : "rani",
            "geoip" : {
              "country_iso_code" : "EG",
              "location" : {
                "lon" : 31.3,
                "lat" : 30.1
              },
              "region_name" : "Cairo Governorate",
              "continent_name" : "Africa",
              "city_name" : "Cairo"
            },
            "event" : {
              "dataset" : "sample_ecommerce"
            }
          }
          ...
        }
      ]
    }
  }
}
```

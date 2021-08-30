---
layout: post
title: elasticsearch template
description: elasticsearch
date:   2021-08-30 23:11:00 +0530
categories: elasticsearch
tags: [ Elasticsearch , template ]
author: easywaldo
comments: true
---
### 엘라스틱서치 템플릿 예제
인덱스를 RDB 의 테이블에 비유할 수 있다고 하면
템플릿은 인덱스가 생성이 될 때 인덱스를 구성하는 방법을 정의하는 문서라고 할 수 있다.
(번역기를 참고하여 문서를 작성한 것이라 잘못된 설명일 수 있음)

### 템플릿 종류
Elasticsearch 7.8 버전에서 부터 소개가 되는 구성가능한 템플릿이라 불리는 Component Template 가 있으며 기존에 템플릿이라 불리던 개념으로서 Index Template 이 있다.

### 레거시 템플릿
아래와 같이 레거시 템플릿을 만들 수 있다.
> https://www.elastic.co/guide/en/elasticsearch/reference/7.9/indices-templates-v1.html

```json
POST _template/product_template
{
  "index_patterns": ["product*"],
  "template": {
    "settings": {
      "number_of_shards": 1
    },
    "mappings": {
      "properties": {
        "brand_name": { "type": "text" },
        "product_name": { "type": "text" }
      }
    },
    "aliases": {
      "mydata": { }
    }
  },
  "version": 1
}
```


### Component Template (구성가능한 템플릿)
> https://www.elastic.co/guide/en/elasticsearch/reference/7.9/index-templates.html

```json
PUT _component_template/component_template1
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        }
      }
    }
  }
}

PUT _component_template/other_component_template
{
  "template": {
    "mappings": {
      "properties": {
        "ip_address": {
          "type": "ip"
        }
      }
    }
  }
}
```

### Index Template
인덱스 템플릿은 컴포넌트 템플릿으로 구성이 가능하며 컴포넌트 템플릿 없이도 생성이 가능하다.

```json
PUT _index_template/template_1
{
  "index_patterns": ["te*", "bar*"],
  "template": {
    "settings": {
      "number_of_shards": 1
    },
    "mappings": {
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z yyyy"
        }
      }
    },
    "aliases": {
      "mydata": { }
    }
  },
  "priority": 200,
  "composed_of": ["component_template1", "other_component_template"],
  "version": 3,
  "_meta": {
    "description": "my custom"
  }
}
```


### 템플릿을 활용한 일자별 색인
아래와 같이 템플릿을 만든 다음 각각 다른 색인에 문서를 넣을 수 있다.
```json
PUT _template/product-*
{
  "index_patterns": ["product-*"],
  "template": {
    "settings": {
      "number_of_shards": 1
    },
    "mappings": {
      "properties": {
        "brand_name": { "type": "text" },
        "product_name": { "type": "text" }
      }
    },
    "aliases": {
      "mydata": { }
    }
  },
  "version": 1
}

PUT /product-1/_doc/1
{
  "brand_name" : "awesome company",
  "product_name" : "great bag"
}

PUT /product-2/doc/2
{
  "brand_name" : "wow company",
  "product_name" : "wow bag"
}

PUT /product-3/doc/3
{
  "brand_name" : "wow company",
  "product_name" : "wow bag"
}

PUT /product-4/doc/4
{
  "brand_name" : "wow company",
  "product_name" : "wow bag"
}


/// 결과 리스트
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 4,
    "successful" : 4,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "product",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "brand_name" : "best company",
          "product_name" : "good shoes"
        }
      },
      {
        "_index" : "product-1",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "brand_name" : "good company",
          "product_name" : "gold bag"
        }
      },
      {
        "_index" : "product-1",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "brand_name" : "awesome company",
          "product_name" : "great bag"
        }
      },
      {
        "_index" : "product-2",
        "_type" : "doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "brand_name" : "question company",
          "product_name" : "question bag"
        }
      },
      {
        "_index" : "product-3",
        "_type" : "doc",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "brand_name" : "wow company",
          "product_name" : "wow bag"
        }
      }
    ]
  }
}

```
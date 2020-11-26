---
title: 使用elasticsearch-analysis-ik分词器实现提取英文词根
tags:  elasticsearch
categories: elasticsearch
---

* TOC
{:toc}


## 简介

> ik分词器作为一个中文的分词器，但是对一些英文的分词不是很友好。比如 I have some apples。正确的分词应该是吧apples的词根给找到也就是apple。但是实际上ik不做特殊处理的话是无法拿到apple的词根的。

## 创建自定义分词器

```json
PUT /index-test-english
{
  "settings": {
    "analysis": {
      "analyzer": {
        "es_ik": {
          "tokenizer":"ik_max_word",
          "type": "custom",
          "filter": [
            "stemmer"
          ]
        }
      }
    }
  }
}
```

## 测试分词效果

### 输入

```
GET index-test-english/_analyze
{
  "text": ["I have two dogs"],
  "analyzer": "es_ik"
}
```

### 输出

```
{
  "tokens" : [
    {
      "token" : "i",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "ENGLISH",
      "position" : 0
    },
    {
      "token" : "have",
      "start_offset" : 2,
      "end_offset" : 6,
      "type" : "ENGLISH",
      "position" : 1
    },
    {
      "token" : "two",
      "start_offset" : 7,
      "end_offset" : 10,
      "type" : "ENGLISH",
      "position" : 2
    },
    {
      "token" : "dog",
      "start_offset" : 11,
      "end_offset" : 15,
      "type" : "ENGLISH",
      "position" : 3
    }
  ]
}
```

## 代码实操

1、创建索引并且使用自定义分词器

```
PUT /index-test-english
{
  "settings": {
    "analysis": {
      "analyzer": {
        "es_ik": {
          "tokenizer":"ik_max_word",
          "type": "custom",
          "filter": [
            "stemmer"
          ]
        }
      }
    }
  },
  "mappings" : {
      "properties" : {
        "content" : {
          "type" : "text",
          "analyzer" : "es_ik",
          "search_analyzer" : "ik_smart"
        },
        "postDate" : {
          "type" : "keyword"
        },
        "user" : {
          "type" : "keyword"
        }
      }
    }
}
```

2、插入数据

```
    @Test
    public void insertDocumentToEs2() throws IOException {
        String index="index-test-english";
        List<String> documents= Arrays.asList("I have two dogs"
        );
        int i=11;
        for (String document : documents) {
            EsDocumentUtils.addDocument(index,i+"",document);
            i++;
        }
    }
```

3、查询

```
GET index-test-english/_search
{
  "query": {
    "match": {
      "content": "dog"
    }
  }
}
```

## 参考文档

```
https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html
```

```
1https://github.com/medcl/elasticsearch-analysis-ik
```

```
https://www.jianshu.com/p/8f91a375420a
```


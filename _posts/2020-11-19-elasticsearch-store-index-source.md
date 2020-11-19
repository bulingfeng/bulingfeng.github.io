---
title: elasticsearch中_source,store, index, _all, 理解
tags:  elasticsearch
categories: elasticsearch
---

* TOC
{:toc}


## 1、简介

> elasticsearch中为了实现某些需求，比如节约存储资源，防止某个字段被搜索，只想具体某些字段储存到到elasticsearch中等等。那么就可以通过设置mapping的时候，设置不同的属性来满足要求。

## 2、属性介绍

>关键字高亮实质上是根据倒排记录中的词项偏移位置，找到关键词，加上前端的高亮代码。这里就要说到store属性，store属性用于指定是否将原始字段写入索引，默认取值为no。如果在Lucene中，高亮功能和store属性是否存储息息相关，因为需要根据偏移位置到原始文档中找到关键字才能加上高亮的片段。在Elasticsearch，因为_source中已经存储了一份原始文档，可以根据_source中的原始文档实现高亮，在索引中再存储原始文档就多余了，所以Elasticsearch默认是把store属性设置为no。

| 属性名  | 默认值 | 作用                                                         | 备注 |
| ------- | ------ | ------------------------------------------------------------ | ---- |
| _source | true   | 是否存储原始文档                                             |      |
| store   | no     | 字段是否存储原文到索引                                       |      |
| index   | true   |                                                              |      |
| _all    | false  | _all字段开启适用于不指定搜索某一个字段，根据关键词，搜索整个文档内容 |      |

## 3、样例

### A、_source

**创建测试索引**

```JSON
PUT index-2020-source
{
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "user":{
        "type": "keyword"
      },
      "content":{
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

**导入测试数据**

```java
	@Test
    public void insertDocumentToEs() throws IOException {
        String index="index-2020-source";
        List<String> documents= Arrays.asList("谷歌地图之父跳槽facebook",
                "谷歌地图之父加盟facebook",
                "谷歌地图创始人拉斯离开谷歌加盟facebook",
                "谷歌地图之父跳槽facebook与wave项目取消有关",
                "谷歌地图之父拉斯加盟社交网站facebook"
                );
        int i=11;
        for (String document : documents) {
            EsDocumentUtils.addDocument(index,i+"",document);
            i++;
        }
    }
```

**查询测试**

```
GET index-2020-source/_search
{
  "size": 20, 
  "query": {
    "match": {
      "content": "加盟"
    }
  }
}
```

**总结**

> 当设置_source中的enable=fasle。则不会储存原文。但是通过搜索关键字依然能搜到对应的数据。只是返回的都是id，而没有原始字段。说明elasticsearch没有存原文，但是存储了倒排的元数据。

**使用场景**

> 1：由于elasticseach默认只会返回id，当设置_source中的enable=fasle时，则需要根据该文档id去倒排索引中取field字段数据，效率比较低下。
>
> 2：当设置_source中的enable=true时，则直接可以通过id直接检索source中的field数据，而不用去倒排索引中去对应数据。
>
> 3：_source可能会造成索引占用空间。可以采用只存储某些必须的原始字段，而无需存储原文的字段则无需存储原文，只需存倒排索引即可。通过设置includes和excludes属性即可。

### B、store

**创建索引**

```
PUT index-2020-store
{
  "mappings": {
    "properties": {
      "user":{
        "type": "keyword",
        "store":false
      },
      "content":{
        "type": "text",
        "store": false, 
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

**插入数据**

```
    @Test
    public void insertDocumentToEs() throws IOException {
        String index="index-2020-store";
        List<String> documents= Arrays.asList("谷歌地图之父跳槽facebook",
                "谷歌地图之父加盟facebook",
                "谷歌地图创始人拉斯离开谷歌加盟facebook",
                "谷歌地图之父跳槽facebook与wave项目取消有关",
                "谷歌地图之父拉斯加盟社交网站facebook"
                );
        int i=11;
        for (String document : documents) {
            EsDocumentUtils.addDocument(index,i+"",document);
            i++;
        }
    }
```

**查询测试**

```
GET index-2020-store/_search
{
  "size": 20, 
  "query": {
    "match": {
      "content": "加盟"
    }
  }
}
```

**查询结果**

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 0.636667,
    "hits" : [
      {
        "_index" : "index-2020-store",
        "_type" : "_doc",
        "_id" : "12",
        "_score" : 0.636667,
        "_source" : {
          "user" : "bulingfeng",
          "content" : "谷歌地图之父加盟facebook"
        }
      },
      {
        "_index" : "index-2020-store",
        "_type" : "_doc",
        "_id" : "15",
        "_score" : 0.51277506,
        "_source" : {
          "user" : "bulingfeng",
          "content" : "谷歌地图之父拉斯加盟社交网站facebook"
        }
      },
      {
        "_index" : "index-2020-store",
        "_type" : "_doc",
        "_id" : "13",
        "_score" : 0.4673073,
        "_source" : {
          "user" : "bulingfeng",
          "content" : "谷歌地图创始人拉斯离开谷歌加盟facebook"
        }
      }
    ]
  }
}

```

**问题**

> 发现虽然设置store=false。但是通过查询依然能查询到原始文档数据。由于_source是默认存储原文的，所以猜测是__source存储原文的优先级高于store。所以只有设置__source为false。然后再设置store=true才可以生效。废话不说开始验证。

**新建索引**

```
PUT index-2020-store
{
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "user":{
        "type": "keyword",
        "store":true
      },
      "content":{
        "type": "text",
        "store": true, 
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

**插入数据和上面一样(这里就不复制粘贴了)**

**查询**

```
GET index-2020-store/_search
{
  "size": 20, 
  "query": {
    "match": {
      "content": "加盟"
    }
  }
}
```

**查询结果**

```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 0.636667,
    "hits" : [
      {
        "_index" : "index-2020-store",
        "_type" : "_doc",
        "_id" : "12",
        "_score" : 0.636667
      },
      {
        "_index" : "index-2020-store",
        "_type" : "_doc",
        "_id" : "15",
        "_score" : 0.51277506
      },
      {
        "_index" : "index-2020-store",
        "_type" : "_doc",
        "_id" : "13",
        "_score" : 0.4673073
      }
    ]
  }
}

```

**疑问**

> 发现即使user和content属性的store都设置为true。依然没有查询出来原文。但是通过分词依然能够查询得到。所以可以得出结论:
>
> 1：source为true时的优先级高于store。当source默认存储原文时候，store设置false会无效。
>
> 2：当source设置为false，即使store属性设置为true。依然不会存储原文。疑问，那么这些数据存到哪里了呢？

### C：index

**创建索引**

```
PUT index-2020-index
{
  "mappings": {
    "properties": {
      "user":{
        "type": "keyword",
        "index": false
      },
      "content":{
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

**插入数据**

```
    @Test
    public void insertDocumentToEs() throws IOException {
        String index="index-2020-index";
        List<String> documents= Arrays.asList("谷歌地图之父跳槽facebook",
                "谷歌地图之父加盟facebook",
                "谷歌地图创始人拉斯离开谷歌加盟facebook",
                "谷歌地图之父跳槽facebook与wave项目取消有关",
                "谷歌地图之父拉斯加盟社交网站facebook"
                );
        int i=11;
        for (String document : documents) {
            EsDocumentUtils.addDocument(index,i+"",document);
            i++;
        }
    }
```

**查询-1**

```
GET index-2020-index/_search
{
  "size": 20, 
  "query": {
    "term": {
      "content": "加盟"
    }
  }
}
```

**返回数据-1**

```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 0.636667,
    "hits" : [
      {
        "_index" : "index-2020-index",
        "_type" : "_doc",
        "_id" : "12",
        "_score" : 0.636667,
        "_source" : {
          "user" : "bulingfeng",
          "content" : "谷歌地图之父加盟facebook"
        }
      },
      {
        "_index" : "index-2020-index",
        "_type" : "_doc",
        "_id" : "15",
        "_score" : 0.51277506,
        "_source" : {
          "user" : "bulingfeng",
          "content" : "谷歌地图之父拉斯加盟社交网站facebook"
        }
      },
      {
        "_index" : "index-2020-index",
        "_type" : "_doc",
        "_id" : "13",
        "_score" : 0.4673073,
        "_source" : {
          "user" : "bulingfeng",
          "content" : "谷歌地图创始人拉斯离开谷歌加盟facebook"
        }
      }
    ]
  }
}
```

**查询-2**

```
GET index-2020-index/_search
{
  "size": 20, 
  "query": {
    "term": {
      "user": "bulingfeng"
    }
  }
}
```

**返回数据-2**

```
{
  "error": {
    "root_cause": [
      {
        "type": "query_shard_exception",
        "reason": "failed to create query: {\n  \"term\" : {\n    \"user\" : {\n      \"value\" : \"bulingfeng\",\n      \"boost\" : 1.0\n    }\n  }\n}",
        "index_uuid": "Boausg9fTT2nG7NpwbPWcg",
        "index": "index-2020-index"
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "index-2020-index",
        "node": "K3hTD0scSlW0U3QoZv8QpQ",
        "reason": {
          "type": "query_shard_exception",
          "reason": "failed to create query: {\n  \"term\" : {\n    \"user\" : {\n      \"value\" : \"bulingfeng\",\n      \"boost\" : 1.0\n    }\n  }\n}",
          "index_uuid": "Boausg9fTT2nG7NpwbPWcg",
          "index": "index-2020-index",
          "caused_by": {
            "type": "illegal_argument_exception",
            "reason": "Cannot search on field [user] since it is not indexed."
          }
        }
      }
    ]
  },
  "status": 400
}
```

返回报错中包含**"reason": "Cannot search on field [user] since it is not indexed."**。说明该字段不能被索引。

**总结**

> 本项目测试使用的elasticseach版本为7.3.1。有博客说index有三种属性分别为analyzed,not_analyzed,no。而7.X版本中已经没有该选项，只有true和false两种选项。index=true可以被搜索，index=false则不可以被搜索。7.X对于分词有专门的关键字，分别为:analyzer,search_analyzer。

**_all**
> 这个参数已经在在es 6.0+已经弃用。具体使用请参考以下文档。
>
>https://stackoverflow.com/questions/57516759/in-elasticsearch-7-how-do-you-search-all-text-fields

## 参考文档

>https://www.cnblogs.com/niutao/p/10909137.html
>
>https://blog.csdn.net/u014589856/article/details/76418188?utm_medium=distribute.pc_relevant.none-task-blog-title-3&spm=1001.2101.3001.4242
>
>https://queirozf.com/entries/what-do-store-index-all-source-mean-in-elasticsearch
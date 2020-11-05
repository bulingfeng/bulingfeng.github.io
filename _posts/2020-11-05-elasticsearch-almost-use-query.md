---
title: elasticsearch常用查询
tags:   elasticsearch查询
categories: elasticsearch
---

* TOC
{:toc}

## 1、非联合查询

| 名称             | 方式     | 是否分词 | 是否指定字段 | 效果                                       |
| ---------------- | -------- | -------- | ------------ | ------------------------------------------ |
| term             | 精确查询 | 否       | 是           | 全匹配                                     |
| match            | 模糊查询 | 是       | 是           | 根据分词模糊查询                           |
| match_phrase     | 顺序匹配 | 是       | 是           | 按照词组进行顺序查询                       |
| multi_match      | 模糊查询 | 是       | 是           | 指定多个字段，只需要某一个字段符合标准即可 |
| queryStringQuery | 全局匹配 | 是       | 否           | 所有字段进行查找                           |

## 2、联合查询

### 简介

> 联合查询是多个条件的复核结果。可以理解为mysql中的where条件。

1. must: 文档必须完全匹配条件
2. 至少满足一个条件以上
3. 文档必须不能匹配上该条件。

### 样例

测试数据如下：

```json
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
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.181Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父跳槽facebook"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.432Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父加盟facebook"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.442Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图创始人拉斯离开谷歌加盟facebook"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.450Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父跳槽facebook与wave项目取消有关"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.457Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父拉斯加盟社交网站facebook"
        }
      }
    ]
  }
}

```

### 进行联合查询

查询条件为:

```json
GET index-test-20201105/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "user": "bulingfeng"
        }
      },
      "must_not": {
        "match": {
          "content": "加盟"
        }
      }
    }
  }
}
```

结果

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
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.181Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父跳槽facebook"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.432Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父加盟facebook"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.442Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图创始人拉斯离开谷歌加盟facebook"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.450Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父跳槽facebook与wave项目取消有关"
        }
      },
      {
        "_index" : "index-test-20201105",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "postDate" : "2020-11-05T11:19:18.457Z",
          "user" : "bulingfeng",
          "content" : "谷歌地图之父拉斯加盟社交网站facebook"
        }
      }
    ]
  }
}

```

### 总结

> 可以看到查询的数据中包含了user=builngfeng，并且查询的content中没有加盟两个字。
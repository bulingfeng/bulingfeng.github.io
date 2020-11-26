---
title: kibana常用查询
tags:   elasticsearch kibana
categories: kibana
---

* TOC
{:toc}

1、无条件查询所有数据（es默认返回10条）

```
GET index/_search
{
  "query": {
    "match_all": {}
  }
}
```

2、测试分词器分词结果

```
GET index/_analyze
{
  "text": "张三是一个男人",
  "analyzer": "ik_smart"  //ik_max_word
}
```

3、查看具体某个索引的mapping信息

```
GET index/_mapping
```

4、查看目前有索引情况

```
GET _cat/indices?v
```

5、match查询

```
GET index/_search
{
  "query": {
    "match": {
      "content": "开始"
    }
  }
}
```

6、term查询

```
GET index/_search
{
  "query": {
    "term": {
      "查询字段": {
        "value": "匹配值"
      }
    }
  }
}
```

7、multi_match

```
多个目标字段中有一个字段能够match到即算成功。
GET index-test-20201111/_search
{
  "query": {
    "multi_match": {
      "query": "开始",
      "fields": ["conent","user"]
    }
  }
}
```

8、bool查询

```
GET index-test-20201111/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "加盟"
          }
        }
      ]
    }
  }
}
```

9、bool查询的复核查询

```
下面的语句含义是(sex=男)&&(score=70 || score=80)
GET index-test-20201111/_search/
{
   "query": {
      "bool": {
         "must": [
            {"term": {"sex": {"value": "男"}}},
            {
                "bool": {
                  "should": [
                     {"term": {"score": {"value": "70"}}},
                     {"term": {"score": {"value": "80"}}}
                  ]
               }
            }
         ]
      }
   }
}
```

10、快速测试analyzer

```java
GET test-index/_analyze
{
  "text": ["2020-11-26"],
  "analyzer": "ik_smart"
}
```


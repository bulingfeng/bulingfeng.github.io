---
title: elasticsearch设置登录账号密码
tags: elasticsearch 
categories: elasticsearch
---

* TOC
{:toc}


## 简介

> elasticsearch7以后默认都装了x-pack。可以免费试用其中的基本组件，其中的密码服务也是免费提供。

## 开始密码验证

> 修改elasticsearch.yml。
>
> 在文件末尾添加以下参数(添加这个参数以后一定要重启elasticsearch服务)：
>
> xpack.security.enabled: true
> 
> xpack.security.transport.ssl.enabled: true

## 设置密码

手动设置密码

> bin/elasticsearch-setup-passwords interactive 

自动设置密码命令

> bin/elasticsearch-setup-passwords auto

## 修改密码命令

```curl
curl -H "Content-Type:application/json" -XPOST -u elastic 'http://127.0.0.1:9200/_xpack/security/user/elastic/_password' -d '{ "password" : "123456" }'
```

## 登录elasticsearch

> 账号：elastic
>
> 密码：123456

## 设置kibana密码
> 修改配置文件kibana.yml
>
> elasticsearch.username: "elastic"
>
> elasticsearch.password: "123456"
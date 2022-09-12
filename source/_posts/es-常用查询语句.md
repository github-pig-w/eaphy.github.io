---
title: es 常用查询语句
tags:
  - null
categories:
  - null
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-09-12 22:38:55
password:
---

```java
#添加数据
PUT /eaphy/student/1
{
  "id":1,
  "age":11,
  "name":"小米",
  "city":"北京",
  "desc":"我是一个北京人"
}
PUT /eaphy/student/2 
{"id":2,"age":12,"city":"南京","name":"小明","desc":"我是一个南京人"}

PUT /eaphy/student/3
{"id":3,"age":13,"city":"上海","name":"张三","desc":"我是一个上海人"}
```
<!-- more -->

```java
#修改数据
POST /eaphy/student/1/_update
{
  "doc": {
    "age":11
  }
}
PUT /eaphy/student/1
{
  "id":1,
  "age":11,
  "name":"小花",
  "city":"北京",
  "desc":"我是一个北京人"
}
# 删除一个type下所有数据
POST /eaphy/student/_delete_by_query?conflicts=proceed
{
  "query":{
    "match_all":{}
  }
}
```

```java
#查询数据  
GET eaphy/student/1

#条件查询，排序，分页
GET eaphy/student/_search
{
  "query": {
    "match": {
      "desc": "是"
    }
  },
  "sort": [
    {
      "age.keyword": {
        "order": "desc"
      }
    }
  ],
  "from": 0, 
  "size": 2
}
# 条件查询，过滤查询
GET eaphy/student/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "小"
          }
        },
        {
          "match": {
            "city": "京"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 12,
            "lte": 20
          }
        }
      }
    }
  },
  "_source": ["name","age"]
}

# 批量查询
GET /eaphy/student/_mget
{
  "docs":[
    {
      "_id":1
    },
        {
      "_id":2
    }
    
    ]
}

GET _mget
{
  "docs":[
    {
      "_index":"eaphy",
      "_type":"student",
      "_id":1
    },
    {
      "_index":"eaphy",
      "_type":"student",
      "_id":2
    }
  ]
}

GET /eaphy/student/_mget
{
  "ids":[1,2]
}

# 批量操作（一般是两行数据，一行定位，一行填充，delete 是一行）
POST _bulk
{"delete":{"_index":"eaphy","_type":"student","_id":1}}
{"create":{"_index":"eaphy","_type":"student","_id":1}}
{"id":1,"age":11,"name":"小米","city":"北京","desc": "我是一个北京人"}
{"update":{"_index":"eaphy","_type":"student","_id":1}}
{"doc":{"age":10}}

```





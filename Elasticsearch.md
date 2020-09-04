# Elasticsearch



# 特点

几乎实时，分布式，按需扩容，支持PB级别

REST风格，面向文档

全文检索、处理同义词、通过相关性给文档评分

生成分析与聚合数据

封装隐藏 Lucene 的复杂性，简单，开箱即用



# 术语

+ 索引 index：保存相关数据的地方，实际上是指向一个或者多个物理 `分片` 的逻辑命名空间。

+ 类别 type

+ 文档 document

+ 定义 settings：索引的定义数据（元数据）。

+ 映射 mapping：==因为我们不能更新已存在的映射。==

+ 分词器 analyzer



+ 关键词 keyword：不会被分词器分割。
+ 停用词 stopwords：的、吗、这等高频词，应当最后筛选，并降低权重。
+ 文本 text：会被分词器分割



+ _index / _type / _id
+ version 版本号
+ scope 权重、分值
+ _source 文档内容



+ 节点 node：一个正在运行的Elasticsearch实例
+ 集群 cluster：n（n >= 1）个实例拥有相同的cluster.name
+ 分片 shards：一个底层的工作单元，数据容器，它仅保存了全部数据中的一部分。==一个分片是一个 Lucene 的实例==，以及它本身就是一个完整的搜索引擎。 我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。==在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。==
+ 主分片 master： 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。
+ 副本分片 replicas：一个副本分片只是一个主分片的拷贝。副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

> 在ElasticSearch中被视为单独的一个索引(index)，在Lucene中可能不止一个。这是因为在分布式体系中，ElasticSearch会用到分片(shards)和备份(replicas)机制将一个索引(index)存储多份。



# 分布式

Elasticsearch 可以横向扩展至数百（甚至数千）的服务器节点，同时可以处理PB级数据。

Elasticsearch 天生就是分布式的，并且在设计时屏蔽了分布式的复杂性。

Elasticsearch 在分布式方面几乎是透明的。并不要求使用者了解分布式系统、分片、集群发现或其他的各种分布式概念。可以使用笔记本上的单节点轻松地运行教程里的程序，但如果你想要在 100 个节点的集群上运行程序，一切依然顺畅。

Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在**后台自动执行**的操作：

- 分配文档到不同的容器 或 `分片` 中，文档可以储存在一个或多个节点中
- 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
- 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
- 将集群中任一节点的请求路由到存有相关数据的节点
- 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复



# REST操作

![截图20207271643](D:\user\01397386\Documents\笔记\Elasticsearch\截图20207271643.png)

| Method |        功能        |
| :----: | :----------------: |
|  GET   |        查询        |
|  POST  | 更新（也可以新增） |
|  PUT   |   新增（会覆盖）   |
| DELETE |        删除        |



## 索引（Index）

```json
// 分词测试
GET _analyze
{
    "analyzer": "keyword/standard/ik_smart",
    "test": "关键词"
}

// 创建名为 blogs 的索引。
// 将分配3个主分片（默认为5）和一份副本（每个主分片拥有一个副本分片）
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}

// 查看索引信息（包括映射mappings，定义settings）
GET indexname

// DELETE 删除啥跟啥
DELETE /indexname
```

```json
// _cat 查看健康状态
GET _cat/health
GET _cat/indices?v
```



## 类别（Type）

#### 添加类别

```json
PUT /indexname
{
    "mappings": {
        "typename": {
            "properties": {
                "name": {
                    "type": "text"
                },
                "age": {
                    "type": "long"
                },
                "birth": {
                    "type": "date"
                }
            }
    	}
    }
}
```

#### 映射

> 每个字段都有一个数据`type`，可以是：
>
> - 简单类型等[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/text.html)，[`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/keyword.html)，[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/date.html)，[`long`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/number.html)， [`double`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/number.html)，[`boolean`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/boolean.html)或[`ip`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/ip.html)。
> - 支持JSON的分层性质的类型，如 [`object`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/object.html)或[`nested`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/nested.html)。
> - 或一种特殊类型的图[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/geo-point.html)， [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/geo-shape.html)或[`completion`](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/search-suggesters-completion.html)。
>
> [数据字段类型](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/mapping-types.html)

```json
PUT my_index 
{
  "mappings": {
    "user": { // type_name
      "_all":       { "enabled": false  }, 
      "properties": { // properties
        "title":    { "type": "text"  }, 
        "name":     { "type": "text"  }, 
        "age":      { "type": "integer" }  
      }
    },
    "blogpost": { 
      "_all":       { "enabled": false  }, 
      "properties": { 
        "title":    { "type": "text"  }, 
        "body":     { "type": "text"  }, 
        "user_id":  {
          "type":   "keyword" 
        },
        "created":  {
          "type":   "date", 
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
```



## 文档（Document）

#### 新增

```json
// 添加文档（指定类别 type，指定 ID），如果没有类别映射 mappings，则自动为此类别 type 添加映射 mappings
// version: 1
// result: create
PUT /indexname/typename/1
{
	"name": "xxx",
    "age": 4
}
// 指定路由，具有相同路由的文档会被自动分配到同一个分片上
// 实际意义：新增一篇 user123 发布的文档
// 查询时可以使用同样方式指定路由来筛选文档
PUT /indexname/typename/1?routing=user123,user345,user456
{
	"name": "xxx",
    "age": 4
}

// POST：添加随机ID文档（必须用 POST，PUT 会异常）
POST /indexname/typename
{
    "name": "xxx",
    "age": 4
}
```



#### 查询

```json
// 查询指定文档
GET indexname/typename/1
// 是否存在指定文档
// 返回 200-OK / 404-Not Found
HEAD indexname/typename/1

// 查询范围内所有文档，反之 match_none
GET /_search
{
    "query": {
        "match_all": {}
    }
}

GET indexname/_search
{
    "query": {
        "match_all": {}
    }
}

GET indexname/typename/_search
{
    "query": {
        // 匹配全部
        "match_all": 
        {
            // 最高分值scope为1.2，默认为1.0
            "boost" : 1.2
        }
    }
}

// 不返回任何文档
GET /_search
{
    "query": {
        "match_none": {}
    }
}
```

```json
// 批量查询
GET _mget
{
    "docs": [
        {
            "_index": "blog",
            "_type": "article1",
            "_id": "1"
        },
        {
            "_index": "blog",
            "_type": "article2",
            "_id": "2"
        }
    ]
}
// 如果在同一 index 下，可以如下简化
GET blog/_mget
{
    "docs": [
        {
            "_type": "article1",
            "_id": "1"
        },
        {
            "_type": "article2",
            "_id": "2"
        }
    ]
}
// 如果 index 和 type 都相同，可以更加简化
GET blog/article/_mget
{
    "docs": [
        { "id": "1" },
        { "id": "2" }
    ]
}
// 或者这样
GET blog/article/_mget
{
    "ids": ["1", "2"]
}
```



#### 搜索

/** 删

> `match` ==匹配==，会使用分词器解析关键词，然后按分词匹配查找（模糊查询、效率低）
>
> `term` ==匹配==，直接使用倒排索引来精确查找指定的词条（精确查询、效率高）
>
> `bool` ==筛选==，会评分
>
> `filter` ==筛选==，执行速度非常快，不会计算相关度（直接跳过了整个评分阶段，命中则为1.0）而且很容易被缓存。
>
> **“筛选”**或者 `must{}`，`should{}` 本身是一个布尔值，**”筛选“**内可以包含一个或多个**“匹配“**（**“筛选”**是一个**“匹配”**数组）
>
> 匹配内的匹配项都是布尔值（可以是某个字段是否等于预期，也可以是一个）

**/



+ Bool

查询对应Lucene中的BooleanQuery，它由一个或者多个子句组成，每个子句都有特定的类型。

+ must

返回的文档必须满足must子句的条件，并且参与计算分值

+ filter

返回的文档必须满足filter子句的条件。但是不会像Must一样，参与计算分值

+ should

返回的文档可能满足should子句的条件。在一个Bool查询中，如果没有must或者filter，有一个或者多个should子句，那么只要满足一个就可以返回。`minimum_should_match`参数定义了至少满足几个子句。

+ must_nout

返回的文档必须不满足must_not定义的条件。

```json
query // 查询，内含一个布尔(bool, term/match* 中一个)
{
    bool: { // 返回布尔，内含多个布尔(must, should, must_not, filter)
        must: { // 返回布尔，内含布尔数组(数组元素为 布尔 或 布尔表达式)
            bool,
            match, // 布尔表达式，内含{"FIELD": "TEXT"}
            term // 布尔表达式，内含{"FIELD": "TEXT"}
        }, 
        should: { // 返回布尔，内含布尔数组(数组元素为 布尔 或 布尔表达式)
            bool,
            match, // 布尔表达式，内含{"FIELD": "TEXT"}
            term // 布尔表达式，内含{"FIELD": "TEXT"}
        }, 
        must_nout: { // 返回布尔，内含布尔数组(数组元素为 布尔 或 布尔表达式)
            bool,
            match, // 布尔表达式，内含{"FIELD": "TEXT"}
            term // 布尔表达式，内含{"FIELD": "TEXT"}
        },
        filter: { // 返回布尔，内含布尔数组(数组元素为 布尔 或 布尔表达式)
            bool,
            term // 布尔表达式，内含{"FIELD": "TEXT"}
        }
    },
}
```



```json
//=================================匹配搜索===============================

GET indexname/typename/_search?q=name:狂神说
//高级匹配
GET indexname/typename/_search
{
    "query": {
        "match": {
            "name": "狂神" // 关键词可以用空格隔开，满足其中一个即可，只是分值scope较低
        },
        
    },
    "_source": ["name", "age"], //结果集只显示这些字段
    "sort": [ //排序，结果集中的score会是null
        {
            "age": {
                "order": "desc"
            }
        }
    ],
    //分页，从0开始
    "from": 0,
    "size": 20
}

// name 是被搜索字段，this is a test是 query 内容，分词后 query 中的任何一个关键字被匹配文档就会被搜索到。
// 如果想查询匹配所有关键词的文档，可以用and操作符连接
GET /_search
{
    "query": {
        "match" : {
            "name" : {
                "query" : "this is a test",
                "operator" : "and"
            }
        }
    }
}
```

```json
//=============================布尔匹配===============================

//must 等价于 &&
//should 等价于 ||
//must_not 等价于 !=
//	|_ match 等价于 ==
//	|_ filter 等价于 范围查询
GET indexname3/_search 或者 GET indexname3/1/_search
{
  "query": {
    "bool": {
        
      // 匹配match，评定分值scope
      "must": [
        {
          "match": {
            "name": "3"
          }
        }
      ],
        
        
      // 过滤filter，过滤掉不匹配的文档document
      "filter": [
        {
          "term": {
            "name": "3"
          }
        },
        {
          "range": {
            "age": {
              "gte": 18
            }
          }
        }
      ]
        
        
    }
  }
}
```



#### 统计

```json
// 统计姓smith的人中，各个爱好的平均年龄
GET /megacorp/employee/_search
{
    "query": {
        "match": {
            "last_name": "smith"
        }
    },
    "aggs": {
        "all_interests": { // 起名字
            "terms": {
                "field": "interests" //统计此字段
            },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```



#### 更新

```json
//PUT添加，自动覆盖（慎用）
//version 会加一
//result: updated
PUT /indexname/typename/1
{
	"name": "xxx",
    "age": 4
}

//POST indexname/typename/id/_update
//增量更新
POST indexname/typename/1/_update
{
    "doc": {
        "name": "newName",
        "age": "4"
    }
}

// 原数据：
PUT test/type1/1
{
    "counter": 1,
    "tags": ["red"]
}
// 更新它，使得 counter 字段的值加上 4，更新如下：
POST test/type1/1/_update
{
    "script": {
        // ctx 是脚本语言中的一个执行对象（context 上下文）
        // 能获取 source, _index, _type, _id, _version, _routing, _parent 等字段
        "inline": "ctx._source.counter += params.count",
        // 使用 painless（Elasticsearch 置脚本语言）更新文档
        "lang": "painless",
        "params": {
            "count": 4
        }
    }
}
// tags 数组增加一个值：blue
POST test/type1/1/_update
{
    "script": {
        "inline": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params": {
            "tag": "blue"
        }
    }
}
// 查询更新
POST blog/_update_by_query
{
    // 对 title 中包含 git 的关键词的文档增加一个 category 字段，值为 git
    "script": {
        "inline": "ctx._source.category = params.category",
        "lang": "painless",
        "params": {
            "category": "git"
        }
    },
    // 查询更新的筛选条件
    "query": {
        "term": { "title": "git" }
    }
}
```



#### 删除

```json
// DELETE 删除啥跟啥
DELETE /indexname
DELETE /indexname/typename/id
// 删除时指定路由
// DELETE /indexname/typename/id?routing=user123

// 查询删除
POST blog/_delete_by_query
{
    "query": {
        "term": {
            "title": "java"
        }
    }
}
POST blog/csdn/_delete_by_query
{
    "query": {
        "match_all": { }
    }
}
```




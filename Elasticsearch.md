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
+ 分片 shards：一个底层的工作单元，数据容器，它仅保存了全部数据中的一部分。一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。 我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。==在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。==
+ 主分片 master： 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。
+ 副本分片 replicas：一个副本分片只是一个主分片的拷贝。副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

```json
// 创建名为 blogs 的索引。
// 将分配3个主分片（默认为5）和一份副本（每个主分片拥有一个副本分片）
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```





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

#### 分词测试

```json
GET _analyze
{
    "analyzer": "keyword/standard/ik_smart",
    "test": "关键词"
}
```

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

#### 增

```json
// 添加文档（指定类别type，指定ID），如果没有类别映射mappings，则自动为此类别type添加映射mappings
// version: 1
// result: create
PUT /indexname/typename/1
{
	"name": "xxx",
    "age": 4
}

// 添加文档（使用默认类别type _doc）
// 相当于 PUT /indexname/_doc/1
PUT /indexname/1
{
	"name": "xxx",
    "age": 4
}

// POST：添加随机ID文档
POST /indexname/typename
{
    "name": "xxx",
    "age": 4
}
```

#### 查

```json
// 查看索引信息（包括映射mappings，定义settings）
GET indexname
// 查询指定文档
GET indexname/typename/1
// 查询所有文档
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
```

```json
//_cat 查看健康状态
GET _cat/health
GET _cat/indices?v
```

#### 搜

> `match` 会使用分词器解析关键词，然后按分词匹配查找（模糊查询、效率低）
>
> `term` 直接使用倒排索引来精确查找指定的词条（精确查询、效率高）
>
> `filter` 过滤器：执行速度非常快，不会计算相关度（直接跳过了整个评分阶段，命中则为1.0）而且很容易被缓存。

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
      "filter": {
        "term": {
          "name": "3"
        },
      "range": {
          "age": {
              "gte": 18
          }
      }
      }
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

#### 改

```json
//PUT添加，自动覆盖
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
```

#### 删

```json
//DELETE 删除啥跟啥
DELETE /indexname
DELETE /indexname/typename/id
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
    "user": { 
      "_all":       { "enabled": false  }, 
      "properties": { 
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




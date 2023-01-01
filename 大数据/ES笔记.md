### 基于ELK技术的ElasticSearch知识整理



## 知识准备工作
### 0、什么是ElasticSearch？它和Lucene以及solr的关系是什么？
这些是自己的知识获取能力，自行百度百科

### 1、下载ElasticSearch的window版，linux版后续说明
自行百度Elastic，然后进到官网进行下载，我的版本是：7.8.0
![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213130146178-876876984.png)


### 2、下载postman
自行百度进行下载
![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213130250673-826968984.png)


### 3、ElasticSearch中的目录解读（ 会tomcat，看到这些目录就不陌生 ）
![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213130728535-1551437737.png)

**进到bin目录下，点击 elasticsearch.bat 文件即可启动 ES 服务**


### 4、ELK技术是什么意思？
- 就图中这三个
![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213152026523-1061175814.png)





### 5、ES非关系型和关系型数据库对应关系

![img](https://img-blog.csdnimg.cn/img_convert/146a779da01f53e7f7a8d53132d3c7cf.png) 



**注意：ES 7.x之后，type已经被淘汰了（ 当然：8.x之后已经完全废弃了 ），其他的没变**

**只要玩ES，那么这个图就要牢牢地记在自己脑海里，后续的名词解释不再过多说明，就是操作这幅图中的东西**



## 1、索引

### 1.1、创建索引

**语法：**

http://ip:port/index_name			如：http://127.0.0.1:9200/create_index		**请求方式：put**

**注：put请求具有幂等性**

指的是： 不管进行多少次重复操作，都是实现相同的结果 

**还具有幂等性的有：put、delete、get**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213120334843-469242057.png) 





### 1.1、获取索引

**语法：**

http://ip:port/index_name			如：http://127.0.0.1:9200/create_index	**请求方式：get**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213120405085-1960098752.png) 



### 1.3、获取ES中的全部索引

http://ip:port/_cat/indices?v		 如：http://127.0.0.1:9200/_cat/indices?v

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124721819-7772121.png) 



### 1.4、删除索引

**语法：**

http://ip:port/index_name			如：http://127.0.0.1:9200/create_index		**注意：请求方式为delete**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124744715-921584943.png) 



## 2、文档_doc

### 2.1、使用post创建doc

**这种方式：是采用ES随机生成id时使用的请求方式**

**注：需要先创建索引，因为：这就类似于关系型数据库中在数据库中的表中 创建数据 **



**语法：**

 http://ip:port/index_name/_doc 		如： http://127.0.0.1:9200/create_index/_doc 		**请求方式：post**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124816281-1881323147.png) 



### 2.2、使用put创建doc - 转幂等性 - 自定义id

**在路径后面加一个要创建的id值即可**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124838383-312742285.png) 



### 2.3、查询文档_doc - 重点

#### 2.3.1、id查询单条_doc

**语法：**

 http://ip:port/index_name/_doc/id			如： http://127.0.0.1:9200/create_index/_doc/100001 		**请求方式：get**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124856484-1117023655.png) 



#### 2.3.2、查询ES中索引下的全部_doc

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search 		**请求方式：get**



**注意：别再body中携带数据了，老衲第一次弄的时候没注意就携带数据了，然后报：**

Unknown key for a VALUE_STRING in [title]

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124933985-1107918723.png) 



**返回的结果：**

```json
{
    "took": 69,		查询花费的时间  毫秒值
    "timed_out": false,			是否超时
    "_shards": {		分片  还没学，先不看
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [		查询出来的 当前索引下的所有_doc文档
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": 1.0,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pty5sX0B_g8Hput6eyzA",
                "_score": 1.0,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "100001",
                "_score": 1.0,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            }
        ]
    }
}
```





### 2.4、文档_doc的修改

#### 2.4.1、全量修改

**原理：利用内容覆盖，重新发一份文档罢了**

**语法：**

 http://ip:port/index_name/_doc/id			如： http://127.0.0.1:9200/create_index/_doc/100001 		**请求方式：post**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213124959330-2026780860.png) 



**获取_doc文档，检验一下**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213125016628-1617914897.png) 



#### 2.4.2、局部修改

**语法：**

 http://ip:port/index_name/_update/id		如： http://127.0.0.1:9200/create_index/_update/100001 		**请求方式：post**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213125033102-888656315.png) 

**检验一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213125048515-2123354970.png) 



### 2.5、文档_doc的删除

**使用delete请求即可**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213125825974-1591863725.png) 



### 2.6、条件查询 -  重点

#### 2.6.1、url携带条件

**语法：**

 http://ip:port/index_name/_search?q=条件字段:值		如： http://127.0.0.1:9200/create_index/_search?q=author:邪 			**请求方式：get**

**注：这种方式不建议用，了解即可吧**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213135331818-1597100229.png) 



- **返回的结果：**

  ```json
  {
      "took": 26,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 2,
              "relation": "eq"
          },
          "max_score": 0.18232156,
          "hits": [
              {
                  "_index": "create_index",
                  "_type": "_doc",
                  "_id": "Pdy1sX0B_g8Hput6sCw6",
                  "_score": 0.18232156,
                  "_source": {
                      "title": "这是第一次学习ES的表数据创建",
                      "author": "紫邪情",
                      "sex": "girl"
                  }
              },
              {
                  "_index": "create_index",
                  "_type": "_doc",
                  "_id": "Pty5sX0B_g8Hput6eyzA",
                  "_score": 0.18232156,
                  "_source": {
                      "title": "这是第一次学习ES的表数据创建",
                      "author": "紫邪情",
                      "sex": "girl"
                  }
              }
          ]
      }
  }
  ```





#### 2.6.2、请求体携带条件 - 推荐使用的一种

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search 		**请求方式：get**



**请求体携带的数据：**

```json
{
    "query": {
        "match":{   // match  匹配、配对
            "author": "邪"		// 条件
        }
    }
}
```

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213151459962-445233088.png) 



**结果返回：**

```json

{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 0.18232156,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": 0.18232156,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pty5sX0B_g8Hput6eyzA",
                "_score": 0.18232156,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            }
        ]
    }
}

```





### 2.7、分页查询 -  重点

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search    	**请求方式：get**

**请求体内容：**

```json

{
    "query":{
        "match_all":{}		// 注意：这里使用的是match_all
    },
    "from": 0,		// 相当于：startNum  这个数字的算法：( 当前页码-1 )*要显示的条数
    "size": 1		// 相当于：pageSize
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213234644329-372369101.png) 



**返回结果：**

```json

{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": 1.0,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            }
        ]
    }
}

```





### 2.8、排序查询 -  重点

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search 		**请求方式：get**



**请求体内容：**

```json

{
    "query":{
        "match_all":{}
    },
    "sort":{		// 排序
        "_id":{		// 根据什么字段排序
            "order":"desc"		// 排序方式  desc降序   asc升序
        }
    }
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213151720017-1089171151.png) 

**返回结果：**

```json

{
    "took": 49,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pty5sX0B_g8Hput6eyzA",
                "_score": null,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                },
                "sort": [
                    "Pty5sX0B_g8Hput6eyzA"
                ]
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": null,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                },
                "sort": [
                    "Pdy1sX0B_g8Hput6sCw6"
                ]
            }
        ]
    }
}

```





### 2.9、多条件查询 - 重点

#### 2.9.1、and查询

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search 		**请求方式：get**



**请求体内容：**

```json
{
	"query":{
		"bool":{
			"must":[		// 就相当于是mysql中的 and拼接条件
				{
					"match":{		// and条件1
						"author":"邪"
					}
				},{
					"match":{		// and条件2
						"_id":"Pdy1sX0B_g8Hput6sCw6"
					}
				}
			]
		}
	}
}
```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213171345524-140540959.png) 



**返回结果：**

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.1823215,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": 1.1823215,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            }
        ]
    }
}
```



#### 2.9.2、or查询

**语法：**



**请求体内容：**

```json

{
	"query":{
		"bool":{
			"should":[		// 对照must，改变的地方
				{
					"match":{
						"author":"邪"
					}
				},{
					"match":{
						"_id":"Pdy1sX0B_g8Hput6sCw6"
					}
				}
			]
		}
	}
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213171448431-989685718.png) 



**返回的结果：**

```json

{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.1823215,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": 1.1823215,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pty5sX0B_g8Hput6eyzA",
                "_score": 0.18232156,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                }
            }
        ]
    }
}

```



### 2.10、范围查询 - 重点

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search  	 **请求方式：get**



**请求体内容：**

```json

{
	"query":{
		"bool":{
			"should":[
				{
					"match":{
						"author":"邪"
					}
				},{
					"match":{
						"title":"一"
					}
				}
			],
			"filter":{		// 就多了这么一个filter range而已
				"range":{
					"id":{
						"gt":1000		// gt >   lt <   在html中见过滴
					}
				}
			}
		}
	}
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213171623871-1014850287.png) 

**返回结果：下面的两条数据是我重新加的**

```json

{
    "took": 34,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 0.0,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Vtz2sn0B_g8Hput64ywM",
                "_score": 0.0,
                "_source": {
                    "id": 10001,		
                    "title": "大王叫我来巡山",
                    "author": "王二麻子"
                }
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "V9z5sn0B_g8Hput6HyyQ",
                "_score": 0.0,
                "_source": {
                    "id": 10002,
                    "title": "论皮包龙是怎么形成的",
                    "author": "波多野结衣"
                }
            }
        ]
    }
}

```



### 2.11、完全匹配 - 精准匹配 - 重点

- 在玩这个之前可以回到前面看一下我前面例子中的：条件查询
- 在那里，老衲做的有一个操作：**只用了一个字符：邪，但是：最后得到了想要的结果**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213185443124-1836613228.png) 



- **上面这种：叫做全文检索**，它是去：我创建的create_index索引下的找寻所有的内容，从而匹配出：author包含“邪”的内容，如果此时自己定的条件是：多个字符的话，那么：底层是把这些字符拆成单个字符了，从而匹配出来的
- **而所谓的完全匹配：就是精准匹配到某条数据，它虽然也会拆成单个字符，但是：<span style="color:blue">查询时，还是整串字符绑定在一起匹配的</span>**
  - 如：查询"紫邪情"，用全文检索就是：紫、邪、情，单独匹配，从而判定，如果此时：使用"紫邪晴"，那么：也会匹配到
  - 但是：如果使用的是完全匹配"紫邪晴"，虽然也会拆分为：紫、邪、晴，可是：它最后去匹配结果时，是使用"紫邪晴"这整个字符串来比对的，所以：最后结果不会匹配到

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search 		**请求方式：get**



**全文检索和完全匹配对比**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213185601641-1289605390.png) 



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213185613205-1949627211.png) 



### 2.12、高亮查询 - 重点

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search    **请求方式：get**



**请求体内容：**

```json

{
    "query":{
        "match":{
            "author":"紫邪情"
        }
    },
    "highlight":{		// 高亮
        "fields":{		// 哪个字段需要高亮
            "author":{}
        }
    }
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213190527413-816398035.png) 



**返回结果：**

```json

{
    "took": 59,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 2.264738,
        "hits": [
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pdy1sX0B_g8Hput6sCw6",
                "_score": 2.264738,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                },
                "highlight": {
                    "author": [
                        "<em>紫</em><em>邪</em><em>情</em>"
                    ]
                }
            },
            {
                "_index": "create_index",
                "_type": "_doc",
                "_id": "Pty5sX0B_g8Hput6eyzA",
                "_score": 2.264738,
                "_source": {
                    "title": "这是第一次学习ES的表数据创建",
                    "author": "紫邪情",
                    "sex": "girl"
                },
                "highlight": {
                    "author": [
                        "<em>紫</em><em>邪</em><em>情</em>"
                    ]
                }
            }
        ]
    }
}

```



### 2.13、组合查询 - 重点

#### 2.13.1、分组

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search 		**请求方式：get**



**请求体内容：**

```json

{
    "aggs": {    // 组合操作标识
        "author_group": {		// 分组之后的名称  随便取
            "terms": {			// 分组标识
                "field": "id"       // 注意：这里分组别用字符串类型的字段，如：author
            }
        }
    },
    "size": 0       // 设定不显示原始数据，否则："hits":{}这个原始数据也会暴露出来
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213195115140-30147017.png) 



**加上size之后的返回结果：**

```json

{
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 4,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []		// 这里的hits就被去掉了
    },
    "aggregations": {
        "author_group": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 10001,
                    "doc_count": 1
                },
                {
                    "key": 10002,
                    "doc_count": 1
                }
            ]
        }
    }
}

```



#### 2.13.2、平均数

**语法：**

 http://ip:port/index_name/_search 		如： http://127.0.0.1:9200/create_index/_search   **请求方式：get**



**请求体内容：**

```json

{
    "aggs":{
        "id_avg":{
            "avg":{
                "field":"id"
            }
        }
    },
    "size":0
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213195216916-1499200460.png) 

**返回结果：**

```json

{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 4,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "id_avg": {
            "value": 10001.5
        }
    }
}

```

**同样的道理：sum总和之类的也可以写的，把标识改为对应的函数名字、field字段即可**



### 2.14、映射关系

- **就相当于：在做mysql数据库中的表结构（ 字段构建嘛：字段名、类型... )**

#### 2.14.1、建立索引

```json

http:127.0.0.1:9200/user	// 请求类型：put

```



#### 2.14.2、建立映射关系

```json

http://127.0.0.1:9200/user/_mapping		// 请求方式：put



// 请求体内容
{
    "properties":{
        "name":{
            "type":"text",
            "index":true
        },
        "age":{
            "type":"keyword",
            "index":false
        },
        "address":{
            "type":"keyword",
            "index":true
        }
    }
}


// 2、查看映射关系
http://127.0.0.1:9200/user/_mapping		// 请求方式：get


// 3、添加数据
http://127.0.0.1:9200/user/_doc		// 请求方式 post

// 请求体内容
{
    "name":"紫邪情",
    "sex":"女滴",
    "age":18,
    "address":"地球村"
}


// 4、查看数据
http://127.0.0.1:9200/user/_search		// 请求方式 post


```



#### 2.14.3、测试 

```json

http://127.0.0.1:9200/user/_search		// 请求方式 get


// 请求体内容
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "name":"邪"
                    }
                },{
                    "match":{
                        "age":18
                    }
                },{
                    "match":{
                        "address":"地球村"
                    }
                }
            ]
        }
    }
}


```



**返回结果：**

```json

{
    "error": {
        "root_cause": [
            {
                "type": "query_shard_exception",
                "reason": "failed to create query: Cannot search on field [age] since it is not indexed.",		// 重点信息在这里，但是：现在先不解答这个错误的原因，继续测试
                "index_uuid": "To8O7VKkR3OM_drdWZQIIA",
                "index": "user"
            }
        ],
        "type": "search_phase_execution_exception",
        "reason": "all shards failed",
        "phase": "query",
        "grouped": true,
        "failed_shards": [
            {
                "shard": 0,
                "index": "user",
                "node": "w6AVu2CHT6OEaXAJmqT8mw",
                "reason": {
                    "type": "query_shard_exception",
                    "reason": "failed to create query: Cannot search on field [age] since it is not indexed.",
                    "index_uuid": "To8O7VKkR3OM_drdWZQIIA",
                    "index": "user",
                    "caused_by": {
                        "type": "illegal_argument_exception",
                        "reason": "Cannot search on field [age] since it is not indexed."
                    }
                }
            }
        ]
    },
    "status": 400
}

```



**上面说：age不允许被index（ 检索 ） ， 那么删掉它，再看效果**

**请求体变为如下：**

```json

{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "name":"邪"
                    }
                },{		// 去掉了age属性
                    "match":{
                        "address":"地球村"
                    }
                }
            ]
        }
    }
}

```

**发现能够获取到结果**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213233045552-1407060124.png) 



**再变一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213233111192-2025374434.png) 





**现在回到前面报的错：failed to create query: Cannot search on field [age] since it is not indexed**

- 为什么报这个错？其实已经告知得很清楚了：field [age] since it is not indexed 属性age不支持被检索，原因：
  -  ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213233201799-1297209821.png) 



**为什么前面使用"地球村"可以查到数据，而使用"地球"就不可以查到？**

-  ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211213233233911-394763768.png) 



#### 2.15.5、text和keyword类型的区别

- text类型支持全文检索和完全查询，即：我搜索时只用字符串中的一个字符照样得到结果
  - **原理：text使用了分词，就是把字符串拆分为单个字符串了**



- keyword类型支持完全查询，即：精确查询，**前提：index不是false**
  - **原理：keyword不支持分词，所以：查询时必须是完全查询（ 所有字符匹配上才可以 ）**













## 3、java操作ES篇 - 重点

### 3.1、摸索java链接ES的流程

- **自行创建一个maven项目**

#### 3.1.1、父项目依赖管理

```xml

	<properties>
        <ES-version>7.8.0</ES-version>
        <log4j-version>1.2.17</log4j-version>
        <junit-version>4.13.2</junit-version>
        <jackson-version>2.13.0</jackson-version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
                <!-- 注意：这里的版本问题，要和下载的window的ES版本一致，甚至后续用linux搭建也是一样的
                          到时用linux时，ES、kibana的版本都有这样的限定
                -->
                <version>${ES-version}</version>
            </dependency>

            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <!-- 注意：这里别搞成了elasticsearch-client
                    这个东西在7.x已经不推荐使用了，而到了8.0之后，这个elasticsearch-client已经完全被废弃了
                 -->
                <artifactId>elasticsearch-rest-high-level-client</artifactId>
                <!-- 同样的，注意版本问题 -->
                <version>${ES-version}</version>
            </dependency>

            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j-version}</version>
            </dependency>

            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit-version}</version>
            </dependency>

            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>${jackson-version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

```





#### 3.1.2、摸索链接流程

##### 3.1.2.1、获取父项目中的依赖

```xml

    <dependencies>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>

```



##### 3.1.2.2、摸索流程

```java


package cn.zixieqing;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName ConnectionTest
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class ConnectionTest {

    /*
     * @Author ZiXieQing
     * @Description // TODO 测试java链接ES
     * @Date  2021/12/14
     * @Param
     * @return
     */

    // 下面这个逻辑，对照shiro中的realm、manager、FilterFactoryBean的逻辑来看（ 没用过的就当我没说^_^ )
    @Test
    public void test() throws IOException {

        // 3、创建HttpHost
        HttpHost host = new HttpHost("127.0.0.1", 9200);// 发现需要：String hostname, int port  这就很简单了涩
      // 当然：这个方法重载中有一个参数scheme  这个是：访问方式 根据需求用http / https都可以  这里想传的话用：http就可以了

        // 2、创建RestClientBuilder 但是：点击源码发现 - 没有构造方法
        // 既然没有，那肯定提供得有和xml版的mybatis加载完xml文件之后的builder之类的，找一下
        RestClientBuilder clientBuilder = RestClient.builder(host);
        // 发现1、有重载；2、重载之中有几个参数，而HttpHost... hosts 这个参数貌似贴近我们想要的东西了，所以建一个HttpHost


        // 1、要链接client，那肯定需要一个client咯，正好：导入得有high-level-client
        RestHighLevelClient esClient = new RestHighLevelClient(clientBuilder);   // 发现需要RestClientBuilder restClientBuilder，那就建

        // 4、测试：只要 esClient. 就可以看到一些很熟悉的方法，可以在这里测试调一下哪些方法，然后去postman中获取数据看一下对不对
        // 这里不多做说明：java链接ES客户端的流程就是上面这样的，不过：这和MySQL数据库链接一样，记得不用了就关闭
        esClient.close();       // 当然：封装之后，这个关闭操作应该放出来，然后在封装的工具中只需要返回这个链接对象即可
    }
}


```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214112543974-1145516732.png) 



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214112555986-1711551616.png) 





### 3.2、java中操作ES索引

#### 3.2.1、向父项目获取自己要的依赖

```xml

    <dependencies>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
        </dependency>
        
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```





#### 3.2.2、封装链接对象

```java

package cn.zixieqing.utile;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;

/**
 * @ClassName ESClientUtil
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class ESClientUtil {

    private static final String HOST = "127.0.0.1";     // 用localhost也行，不过后面用linux就要ip，所以：算在这里养成习惯吧
    private static final Integer PORT = 9200;

    public static RestHighLevelClient getESClient() {

        return new RestHighLevelClient( RestClient.builder( new HttpHost( HOST, PORT ) ) );
    }
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214154758578-1554332469.png) 



#### 3.2.3、创建索引

```java

package cn.zixieqing;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName CreateIndex
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class CreateIndex {

    @Test
    public void createIndexTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 创建索引
        // CreateIndexRequest()  第一个参数：要创建的索引名    第二个参数：请求选项  默认即可
        CreateIndexResponse response = esClient.indices().create(
                new CreateIndexRequest("person"), RequestOptions.DEFAULT );
        

        // 查看是否添加成功  核心方法：isAcknowledged()
        System.out.println( response.isAcknowledged() );


        esClient.close();
    }
}


```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214131111167-1951128108.png) 



**用postman检验一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214131136789-658396446.png) 





#### 3.2.4、查询索引

```java

package cn.zixieqing.index;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.client.indices.GetIndexResponse;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName SearchIndex
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class SearchIndex {

    /*
     * @Author ZiXieQing
     * @Description // TODO 查询索引
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void searchIndexTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 获取索引
        GetIndexResponse response = esClient.indices().get(
                new GetIndexRequest("person"), RequestOptions.DEFAULT );


        // 熟悉GetIndexResponse中的几个api
        System.out.println( "Aliases" + response.getAliases() );
        System.out.println( "Mappings" + response.getMappings() );
        System.out.println( "Settings" + response.getSettings() );      // 这三者在用postman玩的时候，返回结果中都有

        esClient.close();
    }
}


```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214133028511-1416616092.png) 



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214133038569-1596622964.png) 



#### 3.2.5、删除索引

```java

package cn.zixieqing.index;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName DeleteIndex
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class DeleteIndex {

    /*
     * @Author ZiXieQing
     * @Description // TODO 删除索引
     * @Date  2021/12/14
     * @Param
     * @return
     */

    @Test
    public void deleteIndexTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 删除索引
        AcknowledgedResponse response = esClient.indices().delete(
                new DeleteIndexRequest("person"), RequestOptions.DEFAULT );

        // 检验一下：是否删除成功
        System.out.println( response.isAcknowledged() );

        esClient.close();
    }
}


```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214133620216-1028789602.png) 



**用postman再检测一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214133641371-764234266.png) 







### 3.3、java操作ES中的_doc - 重点中的重点

#### 3.3.1、创建doc

- **这里还需要jackson-databind：前面已经导入**
- **同时：为了偷懒，所以把lombok也一起导入了**



- **父项目依赖管理**

```xml

        <lombok-version>1.18.22</lombok-version>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok-version}</version>
        </dependency>
```



- **子项目获取依赖：**

```xml

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>

```



- **测试：**

```xml

package cn.zixieqing.doc;

import cn.zixieqing.entity.UserEntity;
import cn.zixieqing.utile.ESClientUtil;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName InsertDoc
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class InsertDoc {

    /*
     * @Author ZiXieQing
     * @Description // TODO 新增文档_doc，当然下面这个过程利用ESClient封装的思路也可以抽离
     * @Date  2021/12/14
     * @Param
     * @return
     */
    @Test
    public void insertDocTest() throws IOException {

        // 1、获取链接
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        IndexRequest request = new IndexRequest();

        // 选择索引及设置唯一标识
        request.index("user").id("10002");

        // 2、添加数据
        // IndexRequest和IndexResponse这中间就是：做添加doc操作
        UserEntity userEntity = new UserEntity();
        userEntity.setId( "100" ).setName( "紫邪情" ).setSex( "女" );     // lombok链式调用

        // 转json      注：objectMapper是jackson-databind中的，不是ES中的
        String userJson = new ObjectMapper().writeValueAsString( userEntity );

        // 把转成的json字符串存到ES中去
        request.source( userJson , XContentType.JSON);

        // 3、发起请求 获取响应对象
        IndexResponse response = esClient.index( request, RequestOptions.DEFAULT );

        // 看看这个IndexResponse有哪些有用的api
        System.out.println( "响应转态：" + response.getResult() );   // 其他的一点 就可以看到了，都是字面意思

        esClient.close();

    }
}


```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214155932117-1413379852.png) 



**postman检测一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214155949592-2037144055.png) 



**我的测试结构如下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214160122755-1605232909.png) 





#### 3.3.2、修改doc

- **这个修改是指的局部修改，全量修改就不用想了**

```java
package cn.zixieqing.doc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName DeleteDoc
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class UpdateDoc {

    /*
     * @Author ZiXieQing
     * @Description // TODO 删除doc
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void deleteDoc() throws IOException {

        // 1、获取链接对象
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        UpdateRequest request = new UpdateRequest();

        // 获取索引
        request.index("user").id("10002");

        // 2、修改doc数据
        request.doc(XContentType.JSON, "name", "邪公子");

        // 3、发起请求、获得响应对象
        UpdateResponse response = esClient.update(request, RequestOptions.DEFAULT);

        System.out.println( "响应状态为：" + response.getResult() );

        esClient.close();
    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214161822821-1044941200.png) 



**postman检验一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214161841318-83274449.png) 





#### 3.3.3、查询doc

```java

package cn.zixieqing.doc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName GetDoc
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class GetDoc {

    @Test
    public void getDocTest() throws IOException {

        // 1、获取链接对象
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        GetRequest request = new GetRequest();

        request.index("user").id("10002");

        // 2、发起请求、获取响应对象
        GetResponse response = esClient.get(request, RequestOptions.DEFAULT);

        // 3、获取结果  推荐用getSourceAsString()
        String result = response.getSourceAsString();

        System.out.println( "获得的doc为：" + result );

        esClient.close();
    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214163735689-1744925991.png) 



#### 3.3.4、删除doc

```java

package cn.zixieqing.doc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName DeleteDoc
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class DeleteDoc {

    /*
     * @Author ZiXieQing
     * @Description // TODO 删除doc
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void deleteDocTest() throws IOException {

        // 1、获取链接对象
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        DeleteRequest request = new DeleteRequest();

        // 获取索引
        request.index("user").id("10002");

        // 2、做删除操作
        DeleteResponse response = esClient.delete(request, RequestOptions.DEFAULT);

        System.out.println( "响应状态为：" + response.getResult() );

        // 3、关闭链接
        esClient.close();
    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214163802651-1486742156.png) 



**再次获取检验一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214163821199-867450725.png) 





#### 3.4.5、批量新增_doc数据

- **本质：把请求封装了而已，从而让这个请求可以传递各种类型参数，如：删除的、修改的、新增的，这样就可以搭配for循环**



```java

package cn.zixieqing.doc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName BatchDeleteDoc
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class BatchInsertDoc {

    /*
     * @Author ZiXieQing
     * @Description // TODO 批量添加doc数据
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void batchInsertDocTest() throws IOException {

        // 1、获取链接对象
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        BulkRequest request = new BulkRequest();

        // 当然：source的第二个参数都是传个对象，这里为了偷懒，嫖了别人的代码
        request.add( new IndexRequest()
                .index("user")
                .id("520")
                .source( XContentType.JSON, "name", "小紫1") );

        request.add( new IndexRequest()
                .index("user")
                .id("521")
                .source( XContentType.JSON, "name", "小紫2") );

        request.add( new IndexRequest()
                .index("user")
                .id("522")
                .source( XContentType.JSON, "name", "小紫3") );


        // 2、发送请求
        BulkResponse response = esClient.bulk( request, RequestOptions.DEFAULT );

        // 查看执行时间
        System.out.println( response.getTook() );

        esClient.close();

    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214171648016-1693800539.png) 



**postman检验一下：**

```json

http://127.0.0.1:9200/user/_search		请求方式 get


// 返回结果
{
    "took": 585,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "user",
                "_type": "_doc",
                "_id": "520",
                "_score": 1.0,
                "_source": {
                    "name": "小紫1"
                }
            },
            {
                "_index": "user",
                "_type": "_doc",
                "_id": "521",
                "_score": 1.0,
                "_source": {
                    "name": "小紫2"
                }
            },
            {
                "_index": "user",
                "_type": "_doc",
                "_id": "522",
                "_score": 1.0,
                "_source": {
                    "name": "小紫3"
                }
            }
        ]
    }
}

```





#### 3.4.6、批量删除_doc数据

- **本质：把请求封装了而已，从而让这个请求可以传递各种类型参数，如：删除的、修改的、新增的，这样就可以搭配for循环**

```java

package cn.zixieqing.doc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName BarchDeleteDoc
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class BatchDeleteDoc {

    /*
     * @Author ZiXieQing
     * @Description // TODO 批量删除doc数据
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void batchDeleteDoc() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        BulkRequest request = new BulkRequest();

        // 和批量添加相比，变的地方就在这里而已
        request.add(new DeleteRequest().index("user").id("520"));
        request.add(new DeleteRequest().index("user").id("521"));
        request.add(new DeleteRequest().index("user").id("522"));

        BulkResponse response = esClient.bulk(request, RequestOptions.DEFAULT);
        
        System.out.println(response.getTook());

        esClient.close();
    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214171855672-1644830932.png) 



**postman检验一下：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214171910258-431997650.png) 



#### 3.4.7、高级查询 - 重点

##### 3.4.7.1、全量查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName MatchAll
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/

public class MatchAll {

    /*
     * @Author ZiXieQing
     * @Description // TODO 高级查询 - 全量查询  就是基础语法中在请求体内使用match_all那个知识点
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void matchAllTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 全量查询 match_all
        SearchResponse response = esClient.search(  new SearchRequest()
                                                        .indices("user")
                                                        .source(
                                                            new SearchSourceBuilder()
                                                                .query( QueryBuilders.matchAllQuery() )
                                                        ), RequestOptions.DEFAULT );

        // 查看执行了多少时间
        System.out.println( response.getTook() );

        // 把数据遍历出来看一下
        for ( SearchHit data : response.getHits() ) {
            System.out.println( data.getSourceAsString() );
        }


        esClient.close();


        // 上面的看不懂，那就看下面拆分的过程

//        // 1、获取链接对象
//        RestHighLevelClient esClient = ESClientUtil.getESClient();
//
//        // 3、创建SearchRequest对象
//        SearchRequest request = new SearchRequest();
//        request.indices("user");
//
//        // 5、创建SearchSourceBuilder对象
//        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//
//        // 6、进行查询  发现：需要QueryBuilders对象，看源码发现：没有构造，可是：有matchAllQuery()
//        searchSourceBuilder.query( QueryBuilders.matchAllQuery() );
//
//        // 4、调用source()方法获取数据，但是发现：需要SearchSourceBuilder，继续创建
//        request.source( searchSourceBuilder );
//
//        // 2、发送请求  发现：需要SearchRequest  那就建一个
//        SearchResponse response = esClient.search(request, RequestOptions.DEFAULT);
//
//        // 7、获取数据
//        for (SearchHit data : response.getHits()) {
//            System.out.println( data.getSourceAsString() );
//        }
//
//        // 8、关闭链接
//        esClient.close();
    }

}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214192352035-167347418.png) 





##### 3.4.7.2、条件查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName TermQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class TermQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO term条件查询  注意：这里不是说的基础篇中的 filter range 那个条件啊（ 这个条件要求的是查询字段为int类型的 ）
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void termQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 条件查询
        SearchResponse response = esClient.search( new SearchRequest()
                                                    .indices("user")
                                                    .source( new SearchSourceBuilder()
                                                             .query( QueryBuilders.termQuery("_id", "520" ) ) ),  // 对照全量查询：变的就是这里的方法调用
                                                    RequestOptions.DEFAULT );

        for (SearchHit data : response.getHits()) {

            System.out.println( data.getSourceAsString() );
        }

        esClient.close();

    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214192422449-1984041700.png) 





##### 3.4.7.3、分页查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName LimitQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class LimitQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 分页查询    对应基础篇中的from size
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void limitQueryTest() throws IOException {

        // 1、获取链接对象
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 3、创建SearchSourceBuilder
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        // 4、查询出所有的数据
        SearchSourceBuilder sourceBuilder = searchSourceBuilder.query( QueryBuilders.matchAllQuery() );

        // 5、对数据进行分页操作
        sourceBuilder.from(0);
        sourceBuilder.size(2);

        // 2、发送请求
        SearchResponse response = esClient.search( new SearchRequest()
                                                        .indices("user")
                                                        .source( searchSourceBuilder )
                                                    , RequestOptions.DEFAULT );

        // 7、查看数据
        for (SearchHit data : response.getHits()) {

            System.out.println( data.getSourceAsString() );
        }

        // 8、关闭链接
        esClient.close();

    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214192451797-2107474562.png) 





##### 3.4.7.4、排序查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName SortQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class SortQuery {

    @Test
    public void sortQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        SearchSourceBuilder sourceBuilder = searchSourceBuilder.query( QueryBuilders.matchAllQuery() );

        // 排序  以什么字段排序、排序方式是什么( 注意：别犯低级错误啊，用字符串来搞排序 ）
        sourceBuilder.sort("_id", SortOrder.DESC);

        SearchResponse response = esClient.search( new SearchRequest()
                                                        .indices("user")
                                                        .source( searchSourceBuilder ),
                                                   RequestOptions.DEFAULT );

        for (SearchHit data : response.getHits()) {
            System.out.println( data.getSourceAsString() );
        }

        esClient.close();
    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214192519588-1355894956.png) 







##### 3.4.7.5、条件过滤查询

````java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName FilterQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class FilterQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 查询过滤
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void filterQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        // 查询结果只需要什么？
        String[] includes = { "name" };
        // 查询结果不需要什么？
        String[] excludes = {};     // 根据需求自行填充

        searchSourceBuilder.fetchSource( includes,excludes );

        SearchResponse response = esClient.search( new SearchRequest()
                                                        .indices("user")
                                                        .source( searchSourceBuilder )
                                                    , RequestOptions.DEFAULT );

        for (SearchHit data : response.getHits()) {

            System.out.println( data.getSourceAsString() );
        }

        esClient.close();
    }
}

````



**我的数据没弄好，我建的doc中只有一个name，而老衲又懒得加了，所以：这里别让结果把自己搞混了**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214193433725-1118278213.png) 







##### 3.4.7.6、组合查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName UnionQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class UnionQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 组合查询至must查询
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void mustQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 注意：这里产生了改变，是调用的boolQuery()
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();

        // 必须包含什么？
        boolQuery.must( QueryBuilders.matchQuery("author", "邪") );
        boolQuery.must( QueryBuilders.matchQuery("sex", "girl") );
        // 当然：也就有mustNot()不包含什么了

        SearchResponse response = esClient.search( new SearchRequest()
                                                        .source( new SearchSourceBuilder()
                                                                    .query( boolQuery ) )
                                                    , RequestOptions.DEFAULT );


        for (SearchHit data : response.getHits()) {

            System.out.println( data.getSourceAsString() );
        }
        esClient.close();

    }


    /*
     * @Author ZiXieQing
     * @Description // TODO 组合查询之should查询
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void shouldQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();

        // 懒得烧蛇吃  不写了，知道这个should和must一样，复制粘贴多个条件即可
        boolQuery.should( QueryBuilders.matchQuery("title", "是") );

        SearchResponse response = esClient.search( new SearchRequest()
                                                        .source( new SearchSourceBuilder()
                                                                    .query(boolQuery) )
                                                    , RequestOptions.DEFAULT );

        for (SearchHit data : response.getHits()) {
            System.out.println( data.getSourceAsString() );
        }

        esClient.close();
    }
}

```



**must查询的结果：**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214204819239-1418101576.png) 





**should查询的结果**

![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214204836158-971856013.png) 







##### 3.4.7.7、范围查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.query.RangeQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName RangeQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class RangeQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 范围查询  即：基础篇中的filter  range
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void rangeQuery() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("id");

        // 对结果进行处理  gt >  lt <   gte >=     lte  <=
        rangeQuery.gt("10000");

        SearchResponse response = esClient.search( new SearchRequest()
                                                        .source( new SearchSourceBuilder()
                                                                    .query( rangeQuery ) )
                                                    , RequestOptions.DEFAULT );

        for (SearchHit data : response.getHits()) {

            System.out.println( data.getSourceAsString() );

        }

        esClient.close();

    }
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214204922451-1185728432.png) 





##### 3.4.7.8、模糊查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.unit.Fuzziness;
import org.elasticsearch.index.query.FuzzyQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName FuzzyQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class FuzzyQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 模糊查询
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void fuzzyQuery() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 模糊查询
        // fuzziness( Fuzziness.ONE ) 表示的是：字符误差数  取值有：zero、one、two、auto
        // 误差数  指的是：fuzzyQuery("author","网二")这里面匹配的字符的误差嘛
        //                  可以有几个字符不一样 / 多 / 少几个字符？
        FuzzyQueryBuilder fuzzyQuery = QueryBuilders.fuzzyQuery("author","网二").fuzziness( Fuzziness.ONE );

        SearchResponse response = esClient.search( new SearchRequest()
                                                        .source( new SearchSourceBuilder()
                                                                        .query(fuzzyQuery) )
                                                    , RequestOptions.DEFAULT );

        for (SearchHit data : response.getHits()) {

            System.out.println( data.getSourceAsString() );
        }

        esClient.close();

    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214222318736-440110204.png) 





##### 3.4.7.9、高亮查询

```java

package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.query.TermsQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName HighLightQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class HighLightQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 高亮查询 highLight
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void highLightQueryTest() throws IOException {

        // 1、获取链接对象
        RestHighLevelClient esClient = ESClientUtil.getESClient();

        // 3、创建SearchSourceBuilder对象
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        // 4、查询什么数据？
        TermsQueryBuilder termsQuery = QueryBuilders.termsQuery("author", "小紫1");

        // 5、构建高亮
        HighlightBuilder highlightBuilder = new HighlightBuilder();

        // 高亮编写
        highlightBuilder.preTags("<span color='blue'>");    // 构建标签前缀
        highlightBuilder.postTags("</span>");       // 构建标签后缀
        highlightBuilder.field("author");     // 构建高亮字段

        // 6、设置高亮
        searchSourceBuilder.highlighter( highlightBuilder );

        // 7、进行查询
        searchSourceBuilder.query( termsQuery );

        // 2、发送请求、获取响应对象
        SearchResponse response = esClient.search( new SearchRequest().indices("user").source( searchSourceBuilder ) , RequestOptions.DEFAULT);

        // 验证
        System.out.println(response);

        for (SearchHit hit : response.getHits()) {
            System.out.println(hit.getSourceAsString());

            System.out.println( hit.getHighlightFields());
        }


        // 9、关闭链接
        esClient.close();

    }
}


```





##### 3.4.7.10、聚合查询

```java
package cn.zixieqing.docHighLevel.queryDoc;

import cn.zixieqing.utile.ESClientUtil;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;

import java.io.IOException;

/**
 * @ClassName AggQuery
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/

// 聚合查询
public class AggQuery {

    /*
     * @Author ZiXieQing
     * @Description // TODO 最大值查询
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void maxQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        // max("maxId")  这个名字是随便取的  不讲究，就是取个名字而已        联想：有max、就有min、avg、count、sum......
        //                                  注：方法变成term()就是分组了
        // field("_id")  对哪个字段求最大值
        searchSourceBuilder.aggregation( AggregationBuilders.max("maxId").field("id") );

        SearchResponse response = esClient.search(new SearchRequest().source(searchSourceBuilder), RequestOptions.DEFAULT);


        // 检验
        System.out.println(response);

        esClient.close();
    }


    /*
     * @Author ZiXieQing
     * @Description // TODO 分组查询
     * @Date  2021/12/14
     * @Param []
     * @return void
     */
    @Test
    public void groupQueryTest() throws IOException {

        RestHighLevelClient esClient = ESClientUtil.getESClient();
        
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        // 分组
        searchSourceBuilder.aggregation( AggregationBuilders.terms("groupQuery").field("author") );

        SearchResponse response = esClient.search( new SearchRequest().source(searchSourceBuilder), RequestOptions.DEFAULT);
        
        System.out.println(response);

        esClient.close();

    }
}

```



![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211214233511028-2030642111.png) 





## 4、ES高级篇

### 4.1、集群部署

- 集群的意思：就是将多个节点归为一体罢了（ 这个整体就有一个指定的名字了 ）



#### 4.1.1、window中部署集群 - 了解即可

- 把下载好的window版的ES中的data文件夹、logs文件夹下的所有的文件删掉，然后拷贝成三份，对文件重命名

![1640774306459](C:\Users\ZiXieQing\AppData\Roaming\Typora\typora-user-images\1640774306459.png)





- **修改node-1001节点的config/elasticsearch.yml配置文件**

- 这个配置文件里面有原生的配置信息，感兴趣的可以查看，因为现在要做的配置信息都在原生的配置信息里，只是被注释掉了而已，当然：没兴趣的，直接全选删掉，然后做如下配置

  ```yml
  
  # ---------------------------------- Cluster -----------------------------------
  #
  # Use a descriptive name for your cluster:
  # 集群名称  注意：是把多个节点归为一个整体，所以这个集群名字就是各节点归为一体之后的名字
  # 因此：各个节点中这个集群名字也就要一样了
  cluster.name: es-colony
  #
  # ------------------------------------ Node ------------------------------------
  #
  # Use a descriptive name for the node:
  # 节点名称  在一个集群中，这个名字要全局唯一
  node.name: node-1001
  # 是否有资格成为主机节点
  node.master: true
  # 是否是数据节点
  node.data: true
  #
  # ---------------------------------- Network -----------------------------------
  #
  # Set the bind address to a specific IP (IPv4 or IPv6):
  # 当前节点的ip地址
  network.host: 127.0.0.1
  #
  # Set a custom port for HTTP:
  # 当前节点的端口号
  http.port: 1001
  # 当前节点的通讯端口（ 监听端口 ）
  transport.tcp.port: 9301
  # 跨域配置
  http.cors.enabled: true
  http.cors.allow-origin: "*"
  
  ```

  

- **修改node-1002节点的config/elasticsearch.yml配置文件**

```yml

# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称  注意：是把多个节点归为一个整体，所以这个集群名字就是各节点归为一体之后的名字
# 因此：各个节点中这个集群名字也就要一样了
cluster.name: es-colony
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称  在一个集群中，这个名字要全局唯一
node.name: node-1002
# 是否是主机节点
node.master: true
# 是否是数据节点
node.data: true
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
# 当前节点的ip地址
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
# 当前节点的端口号
http.port: 1002
# 当前节点的通讯端口（ 监听端口 ）
transport.tcp.port: 9302
# 当前节点不知道集群中另外节点是哪些涩，所以配置，让当前节点能够找到其他节点
discovery.seed_hosts: ["127.0.0.1:9301"]
# ping请求调用超时时间，但同时也是选主节点的delay time
discovery.zen.fd.ping_timeout: 1m
# 重试次数，防止GC[ 垃圾回收 ]节点不响应被剔除
discovery.zen.fd.ping_retries: 5
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"

```



- **修改node-1003节点的config/elasticsearch.yml配置文件**

```yml

# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称  注意：是把多个节点归为一个整体，所以这个集群名字就是各节点归为一体之后的名字
# 因此：各个节点中这个集群名字也就要一样了
cluster.name: es-colony
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称  在一个集群中，这个名字要全局唯一
node.name: node-1003
# 是否是主机节点
node.master: true
# 是否是数据节点
node.data: true
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
# 当前节点的ip地址
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
# 当前节点的端口号
http.port: 1003
# 当前节点的通讯端口（ 监听端口 ）
transport.tcp.port: 9303
# 当前节点不知道集群中另外节点是哪些涩，所以配置，让当前节点能够找到其他节点
discovery.seed_hosts: ["127.0.0.1:9301","127.0.0.1:9302"]
# ping请求调用超时时间，但同时也是选主节点的delay time
discovery.zen.fd.ping_timeout: 1m
# 重试次数，防止GC[ 垃圾回收 ]节点不响应被剔除
discovery.zen.fd.ping_retries: 5
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"

```



**依次启动1、2、3节点的bin/elasticsearch.bat即可启动集群**



**用postman测试集群**

```json

http://localhost:1001/_cluster/health  # 请求方式：get

# 相应内容
{
    "cluster_name": "es-colony",
    "status": "green",  # 重点查看位置 状态颜色
    "timed_out": false,
    "number_of_nodes": 3,	# 重点查看位置	集群中的节点数量
    "number_of_data_nodes": 3,		# 重点查看位置	集群中的数据节点数量
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100.0
}

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229201657877-1995393640.png) 



**status字段颜色表示：**当前集群在总体上是否工作正常。它的三种颜色含义如下：

1. green：所有的主分片和副本分片都正常运行
2. yellow：所有的主分片都正常运行，但不是所有的副本分片都正常运行
3. red：有主分片没能正常运行



**附加内容：一些配置说明，下面的一些配置目前有些人可能并没有遇到，但是在这里留个印象吧，知道个大概和怎么去找就行了**

官网地址： https://www.elastic.co/guide/en/elasticsearch/reference/current/modules.html 

**1、主节点 [ host区域 ]：**

```json

cluster.name: elastics   #定义集群名称所有节点统一配置
node.name: es-0   # 节点名称自定义
node.master: true  # 主节点,数据节点设置为 false
node.data: false   # 数据节点设置为true
path.data: /home/es/data   
path.logs: /home/es/logs   
bootstrap.mlockall: true        #启动时锁定内存
network.publish_host: es-0
network.bind_host: es-0
http.port: 9200
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping_timeout: 120s
discovery.zen.minimum_master_nodes: 2 #至少要发现集群可做master的节点数，
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["es-0","es-1", "es-2","es-7","es-8","es-4","es-5","es-6"] 
discovery.zen.fd.ping_timeout: 120s
discovery.zen.fd.ping_retries: 6
discovery.zen.fd.ping_interval: 30s
cluster.routing.allocation.disk.watermark.low: 100GB
cluster.routing.allocation.disk.watermark.high: 50GB
node.zone: hot                     #磁盘区域，分为hot和stale，做冷热分离
script.inline: true
script.indexed: true 
cluster.routing.allocation.same_shard.host: true
threadpool.bulk.type: fixed  
threadpool.bulk.size: 32 
threadpool.bulk.queue_size: 100
threadpool.search.type: fixed  
threadpool.search.size: 49 
threadpool.search.queue_size: 10000
script.engine.groovy.inline.aggs: on
index.search.slowlog.threshold.query.warn: 20s
index.search.slowlog.threshold.query.info: 10s
index.search.slowlog.threshold.query.debug: 4s
index.search.slowlog.threshold.query.trace: 1s
index.search.slowlog.threshold.fetch.warn: 2s
index.search.slowlog.threshold.fetch.info: 1600ms
index.search.slowlog.threshold.fetch.debug: 500ms
index.search.slowlog.threshold.fetch.trace: 200ms
index.indexing.slowlog.threshold.index.warn: 20s
index.indexing.slowlog.threshold.index.info: 10s
index.indexing.slowlog.threshold.index.debug: 4s
index.indexing.slowlog.threshold.index.trace: 1s
indices.fielddata.cache.size: 20%
indices.fielddata.cache.expire: "48h"
indices.cache.filter.size: 10%
index.search.slowlog.level: WARN

```



**数据节点 [ stale区域 ]**

```json

cluster.name: elastics  # 集群名字
node.name: es-1     #节点名称
node.master: false      # 不作为主节点，只存储数据
node.data: true         # 做为数据节点
path.data: /data1/es-data,/data2/es-data,/data3/es-data  # 存储目录，可配置多个磁盘
path.logs: /opt/es/logs     # 日志目录
bootstrap.mlockall: true    # 启动时锁定内存
network.publish_host: es-1  # 绑定网卡
network.bind_host: es-1     # 绑定网卡
http.port: 9200             # http端口
discovery.zen.ping.multicast.enabled: false       # 禁用多播，夸网段不能用多播
discovery.zen.ping_timeout: 120s                  
discovery.zen.minimum_master_nodes: 2             # 至少要发现集群可做master的节点数
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["es-0","es-1", "es-2","es-7","es-8","es-4","es-5","es-6"]     # 集群自动发现

# fd 是 fault detection 
# discovery.zen.ping_timeout 仅在加入或者选举 master 主节点的时候才起作用；
# discovery.zen.fd.ping_timeout 在稳定运行的集群中，master检测所有节点，以及节点检测 master是否畅通时长期有用
discovery.zen.fd.ping_timeout: 120s                # 超时时间(根据实际情况调整)
discovery.zen.fd.ping_retries: 6                   # 重试次数，防止GC[垃圾回收]节点不响应被剔除
discovery.zen.fd.ping_interval: 30s                # 运行间隔

# 控制磁盘使用的低水位。默认为85%，意味着如果节点磁盘使用超过85%，则ES不允许在分配新的分片。当配置具体的大小如100MB时，表示如果磁盘空间小于100MB不允许分配分片
cluster.routing.allocation.disk.watermark.low: 100GB      #磁盘限额

# 控制磁盘使用的高水位。默认为90%，意味着如果磁盘空间使用高于90%时，ES将尝试分配分片到其他节点。上述两个配置可以使用API动态更新，ES每隔30s获取一次磁盘的使用信息，该值可以通过cluster.info.update.interval来设置
cluster.routing.allocation.disk.watermark.high: 50GB      # 磁盘最低限额

node.zone: stale                      # 磁盘区域，分为hot和stale，做冷热分离
script.inline: true                   # 支持脚本
script.indexed: true 
cluster.routing.allocation.same_shard.host: true    #一台机器部署多个节点时防止一个分配到一台机器上，宕机导致丢失数据
threadpool.bulk.type: fixed    # 以下6行为设置thread_pool
threadpool.bulk.size: 32 
threadpool.bulk.queue_size: 100
threadpool.search.type: fixed  
threadpool.search.size: 49 
threadpool.search.queue_size: 10000
script.engine.groovy.inline.aggs: on
index.search.slowlog.threshold.query.warn: 20s    # 以下为配置慢查询和慢索引的时间
index.search.slowlog.threshold.query.info: 10s
index.search.slowlog.threshold.query.debug: 4s
index.search.slowlog.threshold.query.trace: 1s
index.search.slowlog.threshold.fetch.warn: 2s
index.search.slowlog.threshold.fetch.info: 1600ms
index.search.slowlog.threshold.fetch.debug: 500ms
index.search.slowlog.threshold.fetch.trace: 200ms
index.indexing.slowlog.threshold.index.warn: 20s
index.indexing.slowlog.threshold.index.info: 10s
index.indexing.slowlog.threshold.index.debug: 4s
index.indexing.slowlog.threshold.index.trace: 1s

```





#### 4.1.2、linux中部署ES

##### 4.1.2.1、部署单机ES



- **准备工作**
  - 1、下载linux版的ES，自行百度进行下载，老规矩，我的版本是：7.8.0
  - 2、将下载好的linux版ES放到自己的服务器中去

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221232681-1230100254.png) 



- **解压文件：命令 tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221305899-1603583964.png) 

- **对文件重命名： 命令 mv elasticsearch-7.8.0 es**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221331397-216765682.png) 



- **创建用户**
  -  **因为安全问题， Elasticsearch 不允许 root 用户直接运行**，所以要创建新用户，在 root 用户中创建新用户 

```json

useradd es		# 新增 es 用户
passwd es		# 为 es 用户设置密码，输入此命令后，输入自己想设置的ES密码即可
userdel -r es	# 如果错了，可以把用户删除了再重新加
chown -R es:es /opt/install/es		# 文件授权		注意：/opt/install/es 改成自己的ES存放路径即可

```





- **修改 config/elasticsearch.yml 配置文件**

```yml

# 在elasticsearch.yml文件末尾加入如下配置
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221510419-1874495087.png) 



**最后：保存   怕你是个大哥不会linux   操作方式 ——— 先按ESC 然后按shift+；分号  最后输入wq即可 **

-  后面懒得说操作方式，linux都不会的话，开什么玩笑



- **修改 /etc/security/limits.conf 文件  命令 vim /etc/security/limits.conf**

```yml

# 在文件末尾中增加下面内容
# 这个配置是：每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536

```



- **修改 /etc/security/limits.d/20-nproc.conf 文件   命令 vim /etc/security/limits.d/20-nproc.conf**

```yml

# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注： * 表示 Linux 所有用户名称

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221628639-412351082.png) 





- **修改 /etc/sysctl.conf 文件**

```yml

# 在文件中增加下面内容
# 一个进程可以拥有的 VMA(虚拟内存区域)的数量,默认值为 65536
vm.max_map_count=655360

```



- **重新加载文件**

```yml
# 命令
sysctl -p

```



- **启动程序 : 准备进入坑中**

```yml

cd /opt/install/es/
# 启动
bin/elasticsearch
# 后台启动
bin/elasticsearch -d 

```

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221756370-574256973.png) 



- 这个错误告知：不能用root用户，所以：**切换到刚刚创建的es用户   命令 su es**
- 然后再次启动程序，**进入下一个坑**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221821932-1082153671.png) 



- 这个错误是因为：启动程序的时候会动态生成一些文件，这和ES没得关系，所以：需要切回到root用户，然后把文件权限再次刷新一下

```yml
# 切换到root用户
su root

# 切换到root用户之后，执行此命令即可
chown -R es:es /opt/install/es

# 再切换到es用户
su es

```



 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221859467-1692441973.png) 



- **再次启动程序**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229221920550-1900530728.png) 

- 吃鸡，这样linux中单机ES就部署成功了



**不过啊，前面这种方式都是low的方式，**有更简单的方式，就是使用docker容器来进行配置，简直不要太简单，虽然：使用docker容器来启动程序有弊端，如：MySQL就不建议放在docker容器中，因为：MySQL是不断地进行io操作，放到docker容器中，就会让io操作效率降低，而ES放到docker中也是同样的道理，但是：可以玩，因为：有些公司其实并没有在意docker的弊端，管他三七二十一扔到docker中



- **如果想要用docker容器进行ES配置，编写如下的docker-compose.yml文件**

```yml

version: "3.1"
services:
  elasticsearch:
    image: daocloud.io/library/elasticsearch:7.9.0  # 注：此网站版本不全，可以直接用管我elasticsearch:7.8.0
    restart: always
    container_name: elasticsearch
    ports:
      - 9200:9200
    environment:
      - JAVA_OPTS=--Xms256m -Xmx1024m
      
```



- **然后启动容器即可**



- **注：使用docker安装需要保证自己的linux中安装了docker和docker-compose，没有安装的话，教程链接如下：**
  -  [centos7安装docker和docker-compose - 紫邪情 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiegongzi/p/15621992.html) 



- **注意：有些人可能还会被防火墙整一下，老衲的防火墙是关了的**

```json

# 暂时关闭防火墙
systemctl stop firewalld
# 永久关闭防火墙
systemctl enable firewalld.service 		# 打开防火墙永久性生效，重启后不会复原
systemctl disable firewalld.service 	# 关闭防火墙，永久性生效，重启后不会复原

```



- **测试是否成功**

```json

# 在浏览器和postman中输入以下指令均可
 http://ip:9200/		# 注：ip是自己服务器的ip    如果是用postman，则：请求方式为 get

```



- **浏览器效果**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229223016789-66269452.png) 

- **注：浏览器访问不了，看看自己服务器开放9200端口没有，别搞这种扯犊子事啊**



- **postman的效果**

 ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211229223035507-352568017.png) 



##### 4.1.2.2、部署集群ES

- **可以选择和windows版的集群搭建一样，复制几份，然后改配置文件，配置都是差不多的**



- **一样的，把linux版的ES解压，重命名**

![1640789894569](C:\Users\ZiXieQing\AppData\Roaming\Typora\typora-user-images\1640789894569.png)



![1640789924427](C:\Users\ZiXieQing\AppData\Roaming\Typora\typora-user-images\1640789924427.png)





- **分发节点**

```json

xsync es-cluster   # es-cluster为解压之后的es名字

# 注：xsync需要单独配置，配置过程如下：

# 1、安装rsync
yum -y install rsync

# 配置hosts 节点服务配置  此配置是在/etc/hosts中进行的
ip name		# 如：192.168.0.100  hadoop


# 2、编写脚本  看自己的/usr/local/bin中是否有xsync文件  都看这里了，那就是第一次弄，肯定没有，所以
# 在/usr/local/bin中新建一个xsync文件  touch xsync
# 编辑内容如下：
#!/bin/sh
# 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
        echo no args...;
        exit;
fi

# 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname
# 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir
# 获取当前用户名称
user=`whoami`
# 循环
for((host=1; host<=2; host++)); do
        echo $pdir/$fname $user@slave$host:$pdir
        echo ==================slave$host==================
        rsync -rvl $pdir/$fname $user@slave$host:$pdir
done
# Note:这里的slave对应自己主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定


# 3、给新建的xsync文件授权
chmod a+x xsync

# 这样就配置成功了
```



- **同样的，root用户不能直接运行，所以创建用户**

```json

useradd es 		# 新增 es 用户
passwd es 		# 为 es 用户设置密码
userdel -r es 	# 如果错了，可以删除再加
chown -R es:es /opt/module/es 		# 给文件夹授权

```



- **编辑 ES文件夹的config/elasticsearch.yml文件，实现集群配置**

```yml

# 加入如下配置，这些配置在网上都有，看不懂的网上比我更详细
# 集群名称
cluster.name: cluster-es
# 节点名称， 每个节点的名称不能重复
node.name: node-1
# ip 地址， 每个节点的地址不能重复
network.host: zixieqing-linux1		# 这个主机名字是在前面使用xsync异步分发时在hosts中配置的名字
# 是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
# head 插件需要这打开这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
# es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举 master
cluster.initial_master_nodes: ["node-1"]
# es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["zixieqing-linux1:9300","zixieqing-linux2:9300","zixieqing-linux3:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
# 集群内同时启动的数据任务个数，默认是 2 个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
# 添加或删除节点及负载均衡时并发恢复的线程个数，默认 4 个
cluster.routing.allocation.node_concurrent_recoveries: 16
# 初始化数据恢复时，并发恢复线程的个数，默认 4 个
cluster.routing.allocation.node_initial_primaries_recoveries: 16


```



- **修改 etc/security/limits.conf  **

```yml

# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536

```



- **修改 /etc/security/limits.d/20-nproc.conf **

```yml

# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
* hard nproc 4096
# 注： * 表示 Linux 所有用户名称

```



-  **修改/etc/sysctl.conf** 

```yml

# 在文件中增加下面内容
vm.max_map_count=655360

```



- **加载文件**

```yml

sysctl -p

```



- **启动软件**

```json

cd /opt/module/es-cluster
# 启动
bin/elasticsearch
# 后台启动
bin/elasticsearch -d

```



**防火墙的问题前面已经提到了**



- **集群验证**

 ![img](https://img-blog.csdnimg.cn/img_convert/0412e37cb5249d1ff0e813ee87f49a50.png) 





### 4.2、巩固核心概念

#### 4.2.1、索引 index

- **所谓索引：类似于关系型数据库中的数据库**
- 但是索引这个东西在ES中又有点东西，它的作用和关系型数据库中的索引是一样的，相当于门牌号，一个标识，旨在：提高查询效率，当然，不是说只针对查询，CRUD都可以弄索引，所以这么一说ES中的索引和关系型数据库中的索引是一样的，就不太类似于关系型中的数据库了，此言差矣！在关系型中有了数据库，才有表结构（ 行、列、类型...... ），而在ES中就是有了索引，才有doc、field.....，因此：这就类似于关系型中的数据库，只是作用和关系型中的索引一样罢了
- **因此：ES中索引类似于关系型中的数据库，作用：类似于关系型中的索引，旨在：提高查询效率，当然：在一个集群中可以定义N多个索引，同时：索引名字必须采用全小写字母**



- 当然：也别忘了有一个倒排索引
  -  关系型数据库通过增加一个**B+树索引**到指定的列上，以便提升数据检索速度。索引ElasticSearch 使用了一个叫做 `倒排索引` 的结构来达到相同的目的 





#### 4.2.2、类型 type

- **这玩意儿就相当于关系型数据库中的表，注意啊：关系型中表是在数据库下，那么ES中也相应的 类型是在索引之下建立的**
- 表是个什么玩意呢？行和列嘛，这行和列有多少N多行和N多列嘛，所以：ES中的类型也一样，可以定义N种类型。同时：每张表要存储的数据都不一样吧，所以表是用来干嘛的？分类 / 分区嘛，所以ES中的类型的作用也来了：就是为了分类嘛。另外：关系型中可以定义N张表，那么在ES中，也可以定义N种类型
- **因此：ES中的类型类似于关系型中的表，作用：为了分类 / 分区，同时：可以定义N种类型，但是：类型必须是在索引之下建立的（ 是索引的逻辑体现嘛 ）**

- **但是：不同版本的ES，类型也发生了变化，上面的解读不是全通用的**
  -  ![image](https://img2020.cnblogs.com/blog/2421736/202112/2421736-20211231180250585-624115802.png) 





#### 4.2.3、文档 document

- **这玩意儿类似管关系型中的行。 一个文档是一个可被索引的基础信息单元，也就是一条数据嘛**



#### 4.2.4、字段field

- **这也就类似于关系型中的列。 对文档数据根据不同属性（ 列字段 ）进行的分类标识 **





#### 4.2.5、映射 mapping

- **指的就是：结构信息 / 限制条件**
- 还是对照关系型来看，在关系型中表有哪些字段、该字段是否为null、默认值是什么........诸如此的限制条件，所以**ES中的映射就是：数据的使用规则设置**





#### 4.2.6、分片 shards - 重要

- **这玩意儿就类似于关系型中的分表**
- 在关系型中如果一个表的数据太大了，查询效率很低、响应很慢，所以就会采用大表拆小表，如：用户表，不可能和用户相关的啥子东西都放在一张表吧，这不是找事吗？因此：需要分表
- 相应的在ES中，也需要像上面这么干，如：存储100亿文档数据的索引，在单节点中没办法存储这么多的文档数据，所以需要进行切割，就是将这整个100亿文档数据切几刀，然后每一刀切分出来的每份数据就是一个分片 （ 索引 ），然后在切开的每份数据单独放在一个节点中，这样切开的所有文档数据合在一起就是一份完整的100亿数据，因此：这个的作用也是为了提高效率
- **创建一个索引的时候，可以指定想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上**



- **分片有两方面的原因：**
  - 允许水平分割 / 扩展内容容量，水平扩充，负载均衡嘛
  - 允许在分片之上进行分布式的、并行的操作，进而提高性能 / 吞吐量



- **注意啊： 当 Elasticsearch 在索引中搜索的时候， 它发送查询到每一个属于索引的分片，然后合并每个分片的结果到一个全局的结果集中 **





#### 4.2.7、副本 Replicas - 重要

- **这不是游戏中的刷副本的那个副本啊。是指：分片的复制品**
- 失败是常有的事嘛，所以：在ES中也会失败呀，可能因为网络、也可能因此其他鬼原因就导致失败了，此时不就需要一种故障转移机制吗，也就是 **创建分片的一份或多份拷贝，这些拷贝就叫做复制分片( 副本 )** 



- **副本（ 复制分片 ）之所以重要，有两个原因：**
  - 在分片 / 节点失败的情况下，**提供了高可用性。因为这个原因，复制分片不与原 / 主要（ original / primary ）分片置于同一节点上是非常重要的**
  - 扩展搜索量 / 吞吐量，因为搜索可以在所有的副本上并行运行





- **多说一嘴啊，分片和副本这两个不就是配套了吗，分片是切割数据，放在不同的节点中（ 服务中 ）；副本是以防服务宕掉了，从而丢失数据，进而把分片拷贝了任意份。这个像什么？不就是Redis中的主备机制吗（ 我说的是主备机制，不是主从复制啊 ，这两个有区别的，主从是一台主机、一台从机，主、从机都具有读写操作；而主备是一台主机、一台从机，主机具有读写操作，而从机只有读操作 ，不一样的啊 ）**





- **不过，有个细节需要注意啊，在Redis中是主备放在一台服务器中，而在ES中，分片和副本不是在同一台服务器中，是分开的，如：分片P1在节点1中，那么副本R1就不能在节点1中，而是其他服务中，不然服务宕掉了，那数据不就全丢了吗**





#### 4.2.8、分配  Allocation

- 前面讲到了分片和副本，对照Redis中的主备来看了，那么对照Redis的主从来看呢？主机宕掉了怎么重新选一个主机？Redis中是加了一个哨兵模式，从而达到的。那么在ES中哪个是主节点、哪个是从节点、分片怎么去分的？就是利用了分配
- **所谓的分配是指： 将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。注意：这个过程是由 master 节点完成的，和Redis还是有点不一样的啊 **





- **既然都说了这么多，那就再来一个ES的系统架构吧**

![1640950708062](C:\Users\ZiXieQing\AppData\Roaming\Typora\typora-user-images\1640950708062.png)

- 其中，**P表示分片、R表示副本**

- **默认情况下，分片和副本都是1，根据需要可以改变**







### 4.3、单节点集群

**这里为了方便就使用window版做演示，就不再linux中演示了**



- **打开前面玩的window版集群的1节点**

 ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101024705309-2049599270.png) 



- **创建索引  把这个索引切成3份（ 切片 ）、每份拷贝1份副本**

```json

http://127.0.0.1:1001/users		# 请求方式：put

# 请求体内容
{
    "settings" : {
        "number_of_shards" : 3,
        "number_of_replicas" : 1
    }
}

```



 ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101024727584-806798957.png) 



- **开始安装head插件，这就是一个可视化界面而已，后续还会用Kibana**

  - 自行到官网下载elasticsearch-head-master，这是用Vue写的
  - 这个插件有两种安装方式，chrome浏览器直接把这个压缩包解压之后，然后把解压的文件夹拖到扩展程序中去，这就成为一个插件，集成到Chrome中去了，也就可以直接用了
  - 还有一种是通过Vue的方式，这种需要保证自己的电脑安装了Node.js，我想都是玩过前后端分离的，也就玩过Vue了，所以这些Vue的配套安装也就有了的 —— 安装Node.js也不难，就官网下载、解压、配置环境变量、然后进到解压的elasticsearch-head-master目录，使用npm install拉取模块，最后使用npm run start就完了。当然npm是国外的，很慢，而使用淘宝的cnpm更快，cnpm安装方式更简单，直接` npm install -g cnpm --registry=https://registry.npm.taobao.org `拉取镜像即可，然后就可以使用cnpm来代替npm，从而执行命令了

  - 由于我用的是Edge浏览器，所以我是采用的Vue方式启动的elasticsearch-head-master，启动效果如下：

    -  ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101024752187-140685767.png) 

    - 访问上图中的地址即可，**但是：这个端口是9100，而我们的ES事9200端口，所以9100访问9200事跨越的，因此：需要对ES设置跨越问题，而这个问题在第一次玩ES集群时就配置了的**
      -  ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101024809993-1807170554.png) 



- **head打开之后就是下图中的样子**

 ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101024825967-1023261513.png) 



- **head链接ES之后就是下图的样子**

 ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101135632160-1156170577.png) 

- 三种颜色再巩固一下：
  - green：所有的主分片和副本分片都正常运行
  - yellow：所有的主分片都正常运行，但不是所有的副本分片都正常运行
  - red：有主分片没能正常运行



- **但是：上述的单节点集群有问题，就是将分片和副本都放在一个节点（ node-1001 ）中了，这样会导致前面说的服务宕掉，数据就没了，做的副本就是无用功**



- **当然：在head中测试时，可能会报master_not_discovered_exception，但是再启动一个节点node-1002之后，发现又可以得吃了，而head界面中的颜色从yellow变成green了，这种情况是因为：原有数据导致的，即前面玩windows版ES集群时有另外的数据在里面，只需要把目录下的data文件夹和logs文件夹“下”，把它的东西删了再启动就可以了**
- **但是啊，这里一是玩的windows版，二是为了玩ES才这么干的，这种方式别轻易干啊，学习阶段还是多上网查一下，有很多解决方案的，这里是玩才搞的**



- 回到正题，怎么解决这个集群问题？





### 4.4、故障转移

- 这个东西其实已经见到了，就是前面说的报master_not_discovered_exception的情况，此时再启动一个节点即可实现故障转移



- **启动node-1002节点**

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101140639845-222765913.png)


- 一样的，可能由于玩windows版时的一些数据导致node-1002节点启动不了，所以删掉data文件夹和logs文件夹下的东西即可

- **刷新head可视化页面**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101141146467-1195376810.png)

- 恢复正常





### 4.5、水平扩容 / 负载均衡

- **启动node-1003节点**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101141712204-188942442.png)


- **刷新head页面**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101142118836-570903667.png)


- 对照前面单节点集群来看，数据就被很好的分开了，这样性能不就提上来了吗？试问是去一个节点上访问数据快还是把数据分开之后，减少压力从而效率快呢？肯定后者嘛

- **但是：如果相应继续扩容呢？即：超过6份数据（ 6个节点，前面讲到过索引切分之后，每一份又是单独的索引、副本也算节点 ），那怎么办？**
	- **首先知道一个点：主分片的数目在索引创建时就已经确定下来了的，这个我们没法改变，这个数目定义了这个索引能够存储的最大数据量（ 实际大小取决于你的数据、硬件和使用场景 ）**
	- **但是，读操作——搜索和返回数据——可以同时被主分片 或 副本分片所处理，<span style="color:blue">所以当你拥有越多的副本分片时，也将拥有越高的吞吐量</span>**
	- **因此：增加副本分片的数量即可**
```json

	http://127.0.0.1:1001/users/_settings		# 请求方式：put

	# 请求体内容
	{
		"number_of_replicas": 2
	}

```

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101143851027-1569491724.png)


- **刷新head页面**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101144041160-878860223.png)







### 4.6、应对故障

- 应对的是什么故障？前面一直在说：服务宕掉了嘛


- **关掉node-1001节点（ 主节点 ）**

- **刷新head页面**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101150701673-264262605.png)



- **但是注意啊：yellow虽然不正常，但是不影响操作啊，就像你看了这个yellow之后，影响你正常发挥吗？只是可能有点虚脱而已，所以对于ES来说也是可以正常查询数据的，只是：效率降低了而已嘛（ 主节点和3个分片都在的嘛 ）**



- **解决这种问题**
	- **开启新节点（ 把node-1001节点启动 ———— 此时它就不是主节点了 ，当初新节点了嘛**
		- ![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101151054051-724296708.png)
			- 这就会报错： unless existing master is discovered 找不到主节点（ 对于启动的集群来说，它现在是新节点涩 ），因此：需要做一下配置修改（ node-1001的config/ElasticSearch.yml ）
```yml

	discovery.seed_hosts: ["127.0.0.1:9302","127.0.0.1:9303"]

```

- 保存开启node-1001节点即可

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101151636379-1911114099.png)

- **刷新head页面**

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101151711260-833507116.png)



- **故障恢复了，所以：这也告知一个问题，配置集群时，最好在每个节点的配置文件中都加上上述的配置，从而节点宕掉之后，重启节点即可（ 不然每次改不得烦死 ），注意：ES版本不一样，这个配置方法不一样的，6.x的版本是用cluster.initial_master_nodes: 来进行配置的**







### 4.7、路由计算和分片控制理论

#### 4.7.1、路由计算

- 路由、路由，这个东西太熟悉了，在Vue中就见过路由router了（ 用来转发和重定向的嘛 ）
- 那在ES中的路由计算又是怎么回事？**这个主要针对的是ES集群中的存数据，试想：你知道你存的数据是在哪个节点 / 哪个主分片中吗（ 副本是拷贝的主分片，所以主分片才是核心 ）？**
	- 当然知道啊，就是那几个节点中的任意一个嘛。娘希匹~这样的骚回答好吗？其实这是由一个公式来决定的
```json

	shard = hash( routing ) % number_of_primary_shards

```

**其中**
- routing是一个任意值，默认是文档的_id，也可以自定义
- number_of_primary_shards 表示主分片的数量（ 如前面切分为了3份 ）
- hash()是一个hash函数嘛

**这就解释了为什么我们要在创建索引的时候就确定好主分片的数量并且永远不会改变这个数量：因为如果数量变化了，那么之前所有路由的值都会无效，文档也再也找不到了**





#### 4.7.2、分片控制

- 既然有了存数据的问题，那当然就有取数据的问题了。**请问：在ES集群中，取数据时，ES怎么知道去哪个节点中取数据（ 假如在3节点中，你去1节点中，可以取到吗？），因此：来了分片控制**

- **其实ES不知道数据在哪个节点中，但是：你自己却可以取到数据，为什么？**
	- 负载均衡涩，轮询嘛。所以这里有个小知识点，就是：协调节点 `coordinating node`，**我们可以发送请求到集群中的任一节点，<span style="color:blue">每个节点都有能力处理任意请求，每个节点都知道集群中任一文档位置</span>，这就是分片控制，而我们发送请求的哪个节点就是：协调节点，它会去帮我们找到我们要的数据在哪里**

- **因此：当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点（ 先知道这样做即可 ）**







### 4.8、数据写流程

- 新建、索引和删除请求都是写操作， 必须在主分片上面完成之后才能被复制到相关的副本分片

- **整个流程也很简单**
	- 客户端请求任意节点（ 协调节点 ）
	- 通过路由计算，协调节点把请求转向指定的节点
	- 转向的节点的主分片保存数据
	- 主节点再将数据转发给副本保存
	- 副本给主节点反馈保存结果
	- 主节点给客户端反馈保存结果
	- 客户端收到反馈结果


![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101175443731-783390209.png)


- **但是：从图中就可以看出来，这套流程完了，才可以做其他事（ 如：才可以去查询数据 ），那我为什么不可以异步呢？就是我只要保证到了哪一个步骤之后，就可以进行数据查询，所以：这里有两个小东西需要了解**


- **在进行写数据时，我们做个小小的配置**





#### 4.8.1、一致性 consistency

- 这玩意就是为了和读数据搭配起来嘛，写入和读取保证数据的一致性呗

- **这玩意可以设定的值如下：**
	- one ：只要主分片状态 ok 就允许执行读操作，这种写入速度快，但不能保证读到最新的更改
	- all：这是强一致性，必须要主分片和所有副本分片的状态没问题才允许执行写操作
	- quorum：**这是ES的默认值啊**, 即大多数的分片副本状态没问题就允许执行写操作。这是折中的方法，write的时候，W>N/2，即参与写入操作的节点数W，必须超过副本节点数N的一半，在这个默认情况下，ES是怎么判定你的分片数量的，就一个公式：**int( ( primary + number_of_replicas ) / 2 ) + 1**
		- **注意：primary指的是创建的索引数量；number_of_replicas是指的在索引设置中设定的副本分片数，如果你的索引设置中指定了当前索引拥有3个副本分片，那规定数量的计算结果为：int( 1 primary + 3 replicas) / 2 ) + 1 = 3，如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此你将无法索引和删除任何文档**
	- realtime request：就是从translog里头读，可以保证是最新的。**但是注意啊：get是最新的，但是检索等其他方法不是( 如果需要搜索出来也是最新的，需要refresh，这个会刷新该shard但不是整个index，因此如果read请求分发到repliac shard，那么可能读到的不是最新的数据，这个时候就需要指定preference=_primar y)**





#### 4.8.2、超时 timeout

- 如果没有足够的副本分片会发生什么？Elasticsearch 会等待，希望更多的分片出现。默认情况下，它最多等待 1 分钟。 如果你需要，你可以使用timeout参数使它更早终止，单位是毫秒，如：100就是100毫秒
- 新索引默认有1个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当number_of_replicas 大于1的时候，规定数量才会执行



**上面的理论不理解、或者感觉枯燥也没事儿，后面慢慢的就理解了，这里只是打个预防针、了解理论罢了**











### 4.9、数据读流程

- 有写流程，那肯定也要说一下读流程嘛，其实和写流程很像，只是变了那么一丢丢而已

- **流程如下：**
	- 客户端发送请求到任意节点（ 协调节点 ）
	- 这里不同，此时协调节点会做两件事：1、通过路由计算得到分片位置，2、还会把当前查询的数据所在的另外节点也找到（ 如：副本 ）
	- 为了负载均衡（ 可能某个节点中的访问量很大嘛，减少一下压力咯 ），所以就会对查出来的所有节点做轮询操作，从而找到想要的数据（ 因此：你想要的数据在主节点中有、副本中也有，但是：给你的数据可能是主节点中的，也可能是副本中的 ———— 看轮询到的是哪个节点中的 ）
	- 节点反馈结果
	- 客户端收到反馈结果

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101194342524-1071075650.png)


**当然：这里有个注意点啊（ 需要结合前面说的一致性理论 ）**

- 在文档( 数据 ）被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的







### 4.10、更新操作流程和批量更新操作流程

#### 4.10.1、更新操作流程
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101203603098-1420838433.png)



- 1、客户端向node 1发送更新请求。
- 2、它将请求转发到主分片所在的node 3 。
- 3、node 3从主分片检索文档，修改_source字段中的JSON，并且尝试重新索引主分片的文档。如果文档已经被另一个进程修改,它会重试步骤3 ,超过retry_on_conflict次后放弃。
- 4、如果 node 3成功地更新文档，它将新版本的文档并行转发到node 1和 node 2上的副本分片，重新建立索引。一旦所有副本分片都返回成功，node 3向协调节点也返回成功，协调节点向客户端返回成功


- **当然：上面有个漏洞，就是万一在另一个进程修改之后，当前修改进程又去修改了，那要是把原有的数据修改了呢？这不就成关系型数据库中的“不可重复读”了吗？**
	- **不会的。因为当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。注意点：这些更改将会“异步转发”到副本分片，并且不能保证它们以相同的顺序到达。 如果 ES 仅转发更改请求，则可能以错误的顺序应用更改，导致得到的是损坏的文档**





#### 4.10.2、批量更新操作流程
- 这个其实更容易理解，单文档更新懂了，那多文档更新就懂了嘛，多文档就请求拆分呗

- **所谓的多文档更新就是：将整个多文档请求分解成每个分片的文档请求，并且将这些请求并行转发到每个参与节点。协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端**

- **原理图的话：我就在网上偷一张了**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101205310579-1860787772.png)


- **其实mget 和 bulk API的模式就类似于单文档模式。区别在于协调节点知道每个文档存在于哪个分片中**



**用单个 mget 请求取回多个文档所需的步骤顺序:**

1、客户端向 Node 1 发送 mget 请求。
2、Node 1为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的点节上。一旦收到所有答复，Node 1 构建响应并将其返回给客户端。可以对docs数组中每个文档设置routing参数。

- bulk API， 允许在单个批量请求中执行多个创建、索引、删除和更新请求

 ![img](https://img-blog.csdnimg.cn/img_convert/83499315a7b8ab81471a88f3e142f0a8.png) 





**bulk API 按如下步骤顺序执行：**

1、客户端向Node 1 发送 bulk请求。
2、Node 1为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。
3、主分片一个接一个按顺序执行每个操作。当每个操作成功时,主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端









### 4.11、再次回顾分片和倒排索引

#### 4.11.1、分片
- 所谓的分片就是：将索引切分成任意份嘛，然后得到的每一份数据都是一个单独的索引

- 分片完成后，我们存数据时，存到哪个节点上，就是通过`shard = hash( routing ) % number_of_primary_shards`得到的

- 而我们查询数据时，ES怎么知道我们要找的数据在哪个节点上，就是通过`协调节点`做到的，它会去找到和数据相关的所有节点，从而轮询（ 所以最后的结果可能是从主分片上得到的，也可能是从副本上得到的，就看最后轮询到的是哪个节点罢了 


#### 4.11.2、倒排索引
- 这个其实在基础篇中一上来说明索引时就提到了，基础篇链接如下：
	- https://www.cnblogs.com/xiegongzi/p/15684307.html

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220101221159368-364868269.png)


- 但是，那只是简单提了一下而已，其实还有三个东西没说明



- 就像图中这里，是将内容（ 关键字 ）拆分了，然后来对应ID，所以：这里还有一种东西：**分词**，后面会接触Kibana，再做详细介绍



##### 4.11.2.1、词条


- **它是指：索引中的最小存储或查询单元**。这个其实很好理解，白话文来讲就是：字或者词组，英文就是一个单词，中文就是字或词组嘛，比如：你要查询的是某一个字或词组，这就是词条呗



- **但是啊，网上数据千千万万，一般的数据结构能够存的下吗？不可能的，所以这里做了文章，采用的是B+树和hash存储（ 如：hashmap）**



##### 4.11.2.2、词典
- 这个就更简单了，就是词条的集合嘛。**字或者词组组成的内容呗**





##### 4.11.2.3、倒排表
- **就是指：关键字 / 关键词在索引中的位置 / 概率，有点类似于数组，你查询数组中某个元素的位置，但是区别很大啊，我只是为了好理解，所以才这么举例子的**







### 4.12、文档搜索

#### 4.12.1、不可变的倒排索引

- 以前的全文检索是将整个文档集合弄成一个倒排索引，然后存入磁盘中，当要建立新的索引时，只要新的索引准备就绪之后，旧的索引就会被替换掉，这样最近的文档数据变化就可以被检索到
- 而索引一旦被存入到磁盘就是不可变的（ 永远都可以修改 ），而这样做有如下的好处：
  - 1、只要索引被读入到内存中了，由于其不变性，所以就会一直留在内存中（ 只要空间足够 ），从而当我们做“读操作”时，请求就会进入内存中去，而不会去磁盘中，这样就减小开销，提高效率了
  - 2、索引放到内存中之后，是可以进行压缩的，这样做之后，也就可以节约空间了
  - 3、放到内存中后，是不需要锁的，如果自己的索引是长期不用更新的，那么就不用怕多进程同时修改它的情况了

- 当然：这种不可变的倒排索引有好处，那就肯定有坏处了“
  - 不可变，不可修改嘛，这就是最大的坏处，当要重定一个索引能够被检索时，就需要重新把整个索引构建一下，这样的话，就会导致索引的数据量很大（ 数据量大小有限制了 ），同时要更新索引，那么这频率就会降低了（ **这就好比是什么呢？关系型中的表，一张大表检索数据、更新数据效率高不高？肯定不高，所以延伸出了：可变索引 ）**



#### 4.12.2、可变的倒排索引

**又想保留不可变性，又想能够实现倒排索引的更新，咋办？**

- 就搞出了`补充索引`，**所谓的补充索引：有点类似于日志这个玩意儿，就是重建一个索引，然后用来记录最近指定一段时间内的索引中文档数据的更新。**这样更新的索引数据就记录在补充索引中了，然后检索数据时，直接找补充索引即可，这样检索时不再重写整个倒排索引了，这有点类似于关系型中的拆表，大表拆小表嘛，**但是啊：每一份补充索引都是一份单独的索引啊，这又和分片很像，可是：查询时是对这些补充索引进行轮询，然后再对结果进行合并，从而得到最终的结果，这和前面说过的读流程中说明的协调节点挂上钩了**



**这里还需要了解一个配套的`按段搜索`，玩过 Lucene 的可能听过。按段，每段也就可以理解为：补充索引，它的流程其实也很简单：**

- 1、新文档被收集到内存索引缓存
- 2、不时地提交缓存
  - 2.1、一个新的段，一个追加的倒排索引，被写入磁盘
  - 2.2、一个新的包含新段名字的提交点被写入磁盘
  - 2.3、磁盘进行同步，所有在文件系统缓存中等待的写入都刷新到磁盘，以确保它们被写入物理文件

- 3、新的段被开启，让它包含的文档可见，以被搜索

- 4、内存缓存被清空，等待接收新的文档



- 一样的，段在查询的时候，也是轮询的啊，然后把查询结果合并从而得到的最终结果
- 另外就是涉及到删除的事情，**段本身也是不可变的， 既不能把文档从旧的段中移除，也不能修改旧的段来进行文档的更新，而删除是因为：是段在每个提交点时有一个.del文件，这个文件就是一个删除的标志文件，要删除哪些数据，就对该数据做了一个标记，从而下一次查询的时候就过滤掉被标记的这些段，从而就无法查到了，这叫逻辑删除（ 当然：这就会导致倒排索引越积越多，再查询时。轮询来查数据也会影响效率 ），所以也有物理删除，它是把段进行合并，这样就舍弃掉被删除标记的段了，从而最后刷新到磁盘中去的就是最新的数据（ 就是去掉删除之后的 ，别忘了前面整的段的流程啊，不是白写的 ）**

















### 4.13、近实时搜索、文档刷新、文档刷写、文档合并

- **ES的最大好处就是实时数据全文检索，但是：ES这个玩意儿并不是真的实时的，而是近实时 / 准实时，原因就是：ES的数据搜索是分段搜索，最新的数据在最新的段中（ 每一个段又是一个倒排索引 ），只有最新的段刷新到磁盘中之后，ES才可以进行数据检索，这样的话，磁盘的IO性能就会极大的影响ES的查询效率，而ES的目的就是为了：快速的、准确的获取到我们想要的数据，因此：降低数据查询处理的延迟就very 重要了，而ES对这方面做了什么操作？**
  - **就是搞的一主多副的方式（ 一个主分片，多个副本分片 ），这虽然就是一句话概括了，但是：里面的门道却不是那么简单的**





**首先来看一下主副操作**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220104003247232-66241898.png)


- **但是：这种去找寻节点的过程想都想得到会造成延时，而延时 = 主分片延时 + 主分片拷贝数据给副本的延时**
- **而且并不是这样就算完了，前面提到了N多次的分段、刷新到磁盘还没上堂呢，所以接着看**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220104012357616-597619654.png)





- **但是：在flush到磁盘中的时候，万一断电了呢？或者其他原因导致出问题了，那最后数据不就没有flush到磁盘吗。因此：其实还有一步操作，把数据保存到另外一个文件中去**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220104012442408-122951128.png)



- 数据放到磁盘中之后，translog中的数据就会清空
- 同时更新到磁盘之后，用户就可以进行搜索数据了

- **注意：**这里要区分一下，数据库中是先更新到log中，然后再更新到内存中，而ES是反着的，是先更新到Segment（ 可以直接认为是内存，因它本身就在内存中 ），再更新到log中

- **可是啊，还是有问题，flush刷写到磁盘是很耗性能的，假如：不断进行更新呢？这样不断进行IO操作，性能好吗？也不行，因此：继续改造（ 在Java的JDBC中我说过的 ———— 没有什么是加一层解决不了的，一层不够，那就再来一层 ）**

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220104010946451-860053547.png)


- 加入了缓存之后，这缓存里面的数据是可以直接用来搜索的，这样就不用等到flush到磁盘之后，才可以搜索了，这大大的提高了性能，而flush到磁盘，只要时间到了，让它自个儿慢慢flush就可以了（ 操作系统缓存中的数据断电不会丢失啊，只会在下一次启动的时候，继续执行而已 ），**上面这个流程也叫：持久化 / 持久化变更**

- **写入和打开一个新段的轻量的过程叫做refresh。默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 ES是近实时搜索：文档的变化并不是立即对搜索可见，但会在一秒之内变为可见**
	- **刷新是1s以内完成的，这是有时间间隙的，因此会造成：搜索一个文档时，可能并没有搜索到，因此：解决办法就是使用refresh API刷新一下即可**
	- **但是这样也伴随一个问题：虽然这种从内存刷新到缓存中看起来不错，但是还是有性能开销的。并不是所有的情况都需要refresh的，假如：是在索引日志文件呢？去refresh干嘛，浪费性能而已，所以此时：你要的是查询速度，而不是近实时搜索，因此：可以通过一个配置来进行改动，从而降低每个索引的刷新频率**
```json
	http://ip:port/index_name/_settings		# 请求方式：put

	# 请求体内容
	{
		"settings": {
			"refresh_interval": "60s"
		}
	}

```

- **refresh_interval可以在既存索引上进行动态更新。在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来（ 虽然有点麻烦，但是按照ES这个玩意儿来说，确实需要这么做比较好 ）**
```json

	# 关闭自动刷新
	http://ip:port/users/_settings		# 请求方式：put

	# 请求体内容
	{ 
		"refresh_interval": -1 
	}

	# 每一秒刷新
	http://ip:port/users/_settings		# 请求方式：put
	# 请求体内容
	{ 
		"refresh_interval": "1s" 
	}

```

- 另外：不断进行更新就会导致很多的段出现（ 在内存刷写到磁盘哪里，会造成很多的磁盘文件 ），因此：在哪里利用了文档合并的功能（ 也就是段的能力，合并文档，从而让刷写到磁盘中的文档变成一份 ）









### 4.14、文档分析
- **试想：我们在浏览器中，输入一条信息，如：搜索“博客园紫邪情”，为什么连“博客园也搜索出来了？我要的是不是这个结果涩”**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220104231825619-2002258303.png)

- 这就是全文检索，就是ES干的事情（ 过滤数据、检索嘛 ），但是：它做了哪些操作呢？

**在ES中有一个文档分析的过程，文档分析的过程也很简单：**
- **将文本拆成适合于倒排索引的独立的词条，然后把这些词条统一变为一个标准格式，从而使文本具有“可搜索性”**

- **而这个文档分析的过程在ES是由一个叫做“分析器 analyzer”的东西来做的，这个分析器里面做了三个步骤**
	- **1、字符过滤器：就是用来处理一些字符的嘛，像什么将 & 变为 and 啊、去掉HTML元素啊之类的。**它是文本字符串在经过分词之前的一个步骤，文本字符串是按文本顺序经过每个字符串过滤器从而处理字符串的

	- **2、分词器：见名知意，就是用来分词的，也就是将字符串拆分成词条（ 字 / 词组 ），这一步和Java中String的split()一样的，通过指定的要求，把内容进行拆分，如：空格、标点符号**

	- **3、Token过滤器：这个玩意儿的作用就是 词条经过每个Token过滤器，从而对数据再次进行筛选，如：字母大写变小写、去掉一些不重要的词条内容、添加一些词条（ 如：同义词 ）**

- 上述的内容不理解没事，待会儿会用IK中文分词器来演示，从而能够更直观的看到效果



**在ES中，有提供好的内置分析器、我们也可以自定义、当然还有就是前面说的IK分词器也可以做到（ 而这里重点需要了解的就是IK中文分词器 ）** 


**在演示在前，先玩kibana吧，原本打算放在后面的，但是越早熟悉越好嘛，所以先把kibana说明了**
#### 4.14.1、kibana
- **准备工作：去Elastic官网下载kibana，官网地址如下：**
	- https://www.elastic.co/cn/downloads/?elektra=home&storm=hero
	- 这网站进去会很慢，也可能网不好进不去，那就多刷新几次（ 或者直接搜索Elastic官网再进去也行 ），进入之后就可以在首页看到一个kibana了，**但是需要注意：<span style="color:blue">kibana的版本必须和ES的版本一致</span>，在Java篇中已经说明过了，个人建议：把Windows版和linux版都下载了，玩的时候用Windows版的，linux版后续自己可能会用到**

- **下载好了kibana之后，解压到自己想要的目录（ 注：加压会有点久，因为是用Vue写的，里面有模块module 要是加压快的话，可能还下错了 ），然后点击bin/kibana.bat即可启动kibana**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105012632970-289933603.png)

- 启动之后就是上图中的样子，然后访问图中的地址即可，**第一次进去会有一个选择页面，try / explore，选择explore就可以了，进去之后就是如下界面**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105013048852-1688209141.png)

- 这是英文版，要是没玩过大数据的话，那么里面的一些专业名词根据英文来看根本不知道，所以：**汉化吧，kibana本身就提供得有汉化的功能，只需要改动一个配置即可 —— 就是一个i1bn配置而已**
	- **进入config/kibana.yml，刷到最底部**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105013901463-1104591327.png)


	- 加上上面的信息，然后重启kibana就可以了（ 但是：个人建议，先汉化一段时间，等熟悉哪些名词了，然后再转成英文 ，总之最后建议用英文，一是增加英文词汇量，二是熟悉英文专业词 ）
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105014151449-852838876.png)

	- 汉化成功

**kibana遵循的是rest风格（ get、put、delete、post..... ），具体用法接下来玩分析器和后面都会慢慢熟悉**


#### 4.14.2、内置分析器
##### 4.14.2.1、标准分析器 standard
- **这是根据Unicode定义的单词边界来划分文本，将字母转成小写，去掉大部分的标点符号，从而得到的各种语言的最常用文本选择，<span style="color:blue">另外：这是ES的默认分析器</span>，接下来演示一下**

- **启动ES（ 这是用的单机，即重新解压启动的那种，另外方式也可以玩，但没演示 ）和kibana，打开控制台**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105014716296-1206536683.png)

- **编写指令**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105015151093-584374674.png)

```json

	GET _analyze
	{
	  "analyzer": "standard", # analyzer 分析器  standard 标准分析器
	  "text": "my name is ZiXieQing" # text 文本标识   my name is ZiXieQing 自定义的文本内容
	}


	 # 响应内容
	{
	  "tokens" : [
		{
		  "token" : "my",		# 分词之后的词条
		  "start_offset" : 0,
		  "end_offset" : 2,		# start和end叫偏移量
		  "type" : "<ALPHANUM>",
		  "position" : 0	# 当前词条在整个文本中所处的位置
		},
		{
		  "token" : "name",
		  "start_offset" : 3,
		  "end_offset" : 7,
		  "type" : "<ALPHANUM>",
		  "position" : 1
		},
		{
		  "token" : "is",
		  "start_offset" : 8,
		  "end_offset" : 10,
		  "type" : "<ALPHANUM>",
		  "position" : 2
		},
		{
		  "token" : "zixieqing",
		  "start_offset" : 11,
		  "end_offset" : 20,
		  "type" : "<ALPHANUM>",
		  "position" : 3
		}
	  ]
	}

```

	- **从上图可以看出：所谓标准分析器是将文本通过标点符号来分词的（ 空格、逗号... ，不信可以自行利用这些标点测试一下，观察右边分词的结果 ），同时大写转小写**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105020023125-141039477.png)



##### 4.14.2.2、简单分析器 simple
- **简单分析器是“按非字母的字符分词，例如：数字、标点符号、特殊字符等，会去掉非字母的词，大写字母统一转换成小写”**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105021823253-1892973906.png)


##### 4.14.2.3、空格分析器 whitespace
- **是简单按照空格进行分词，相当于按照空格split了一下，大写字母不会转换成小写**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105021950173-1159454518.png)



##### 4.14.2.4、去词分析器 stop
- **会去掉无意义的词（ 此无意义是指语气助词等修饰性词，补语文：语气词是疑问语气、祈使语气、感叹语气、肯定语气和停顿语气 ），例如：the、a、an 、this等，大写字母统一转换成小写**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105023052858-667836704.png)



##### 4.14.2.5、不拆分分析器 keyword
- **就是将整个文本当作一个词**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105023223725-897621603.png)









#### 4.14.3、IK中文分词器
- 来个实验
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105145147364-1600170909.png)

- 它把我的名字进行拆分了，这不是我想要的，我想要的“紫邪情”应该是一个完整的词，同样道理：**想要特定的词汇，如：ID号、用户名....，这些不应该拆分，而ES内置分析器并不能做到，所以需要IK中文分词器（ 专门用来处理中文的 ）**

- **1、下载IK分词器**
	- https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.8.0
	- **注意：版本对应关系，还是和ES版本对应，https://github.com/medcl/elasticsearch-analysis-ik 这个链接进去之后有详细的版本对应**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105153145884-1511282106.png)
	- **要是感觉github下载慢的话，我把阿里云盘更新了一遍，那里面放出来的包有7.8.0的版本，这里面也有一些其他的东西，有兴趣的下载即可，另外：在这里面有一个chrome-plugin包，这里面有一些chrome的插件，其中有一个fast-github，即：github下载加速器，可以集成到浏览器中，以后下载github的东西就不限速了，云盘链接是：https://www.aliyundrive.com/s/oVC6WWthpUb**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105161254428-1108055410.png)

![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105161624341-733941123.png)

- **fast-github集成到goole之后如下：**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105161731076-1380303432.png)

- 这样以后下载github的东西时，就有一个**加速下载**了，点击即可快速下载，如：
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105162111859-1035334560.png)




- **2、把IK解压到ES/plugins中去，如我的：**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105153337728-1575189913.png)

- **3、重启ES即可（ kibana开着的话，也要关了重启 ），注意观察：重启时会有一个IK加载过程**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105153538766-781620581.png)

- 经过如上的操作之后，IK中文分词器就配置成功了，接下来就来体验一下（ 启动ES和kibana ），主要是为了了解IK中的另外两种分词方式：**ik_max_word和ik_smart**
- **ik_max_word是细粒度的分词，就是：穷尽词汇的各种组成**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105154426087-1329094664.png)


- **ik_smart是粗粒度的分词**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105154611446-2137350220.png)


- **回到前面的问题，“紫邪情”是名字，我不想让它分词，怎么做？上面哪些分词都是在一个“词典”中，所以我们自己搞一个词典即可**
- **1、创建一个.dic文件  dic就是dictionary词典的简写**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105155414310-1843376299.png)

- **2、在创建的dic文件中添加不分词的词组，保存**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105155658747-1397300776.png)

- **3、把自定义的词典放到ik中去，保存**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105155845889-1043411400.png)

- **4、重启ES和kibana**
- **5、测试**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105160054827-345258573.png)

- 可见，现在就把“紫邪情”组成词组不拆分了，前面玩的kibana汉化是怎么做的？和这个的原理差不多


#### 4.14.4、自定义分析器
- 这里还有一个自定义分析器的知识点，这个不了解也罢，有兴趣的自行百度百科了解一下







#### 4.14.5、多玩几次kibana
- 在第一篇高级篇中我边说过：kibana重要，只是经过前面这些介绍了使用之后，并不算熟悉，因此：多玩几次吧
- 另外：就是前面说的kibana遵循rest风格，在ES中是怎么玩的？总结下来其实就下面这些，要上手简单得很，但理论却是一直弄到现在
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105194611033-2003263447.png)

- 现在用kibana来演示几个，其他在postman中怎么弄，换一下即可（ 其实不建议用postman测试，专业的人做专业的事，kibana才是我们后端玩的 ）

- **1、创建索引**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105200834345-1018761313.png)


- **2、查看索引**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105200938111-371770759.png)



- **3、创建文档（ 随机id值，想要自定义id值，在后面加上即可 ）**


- **4、删除索引**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105201030012-825246940.png)

- **5、创建文档（ 自定义id ）**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105201606491-473612050.png)


- **6、查看文档（ 通过id查询 ）**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105201932017-1594955388.png)


- **7、修改文档（ 局部修改 ）**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105202324831-413531788.png)

	- **验证一下：**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105202406803-1472899659.png)

- **8、建字段类型**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105214413765-1715258189.png)




- **其他的也是差不多的玩法，在基础篇中怎么玩，稍微变一下就是kibana的玩法了**




### 4.15、文档控制（ 了解即可 ）
- **所谓的文档控制就是：不断更新的情况，试想：多进程不断去更新文档，会造成什么情况？会把其他人更新过的文档进行覆盖更新了，而ES是怎么解决这个问题的？**
- **就是弄了一个锁来实现的，和Redis一样，也是用的乐观锁来实现的，这个其实没什么好说的，只需要看一下就知道了**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105203825737-375166735.png)

- **上图中·的三个字段就和锁挂钩的，version，版本号嘛，每次更新都会有一个版本号，这样就解决了多进程修改从而造成的文档冲突了（ 必须等到一个进程更新完了，另一个进程才可以更新 ），当然：需要注意旧版本的ES在请求中加上version即可，但是新版本的ES需要使用  if"_seq_no" 和"if_primary_term" 来达到version的效果**













### 4.16、ES的优化
- **ES的所有索引和文档数据都是存储在本地的磁盘中的，所以：磁盘能处理的吞吐量越大，节点就越稳定**

- **要修改的话，是在config/elasticsearch.yml中改动**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105210626346-1575067102.png)





#### 4.16.1、硬件方面
- **1、选用固态硬盘（ 即：SSD ），它比机械硬盘的好是因为：机械硬盘是通过旋转马达的驱动来进行的，所以这就会造成发热、磨损，就会影响ES的效率，而SSD是使用芯片式的闪存来存储数据的，性能比机械硬盘好得多**

- **2、使用RAID 0 （ 独立磁盘冗余阵列 ），它是把连续的数据分散到多个磁盘上存取，这样，系统有数据请求就可以被多个磁盘并行的执行，每个磁盘执行属于它自己的那部分数据请求。这种数据上的并行操作可以充分利用总线的带宽，显著提高磁盘整体存取性能**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105210212220-604356378.png)


- **3、由上面的RAID 0可以联想到另外一个解决方式：使用多块硬盘，也就可以达到同样的效果了（ 有钱就行 ），是通过path data目录配置把数据条分配到这些磁盘上面**


- **4、不要把ES挂载到远程上去存储**







#### 4.16.2、分片策略
- **分片和副本不是乱分配的！分片处在不同节点还可以（ 前提是节点中存的数据多 ），这样就类似于关系型中分表，确实可以算得到优化，但是：如果一个节点中有多个分片了，那么就会分片之间的资源竞争，这就会导致性能降低**


**所以分片和副本遵循下面的原则就可以了**
- **1、每个分片占用的磁盘容量不得超过ES的JVM的堆空间设置（ 一般最大为32G ），假如：索引容量为1024G，那么节点数量为：1024 / 32 = 32左右**

- **2、分片数不超过节点数的3倍，就是为了预防一个节点上有多个分片的情况，万一当前节点死了，那么就算做了副本，也很容易导致集群丢失数据**

- **3、节点数 <= 主节点数 * （ 副本数 + 1 ）**



- **4、推迟分片分配**
- **有可能一个节点宕掉了，但是后面它又恢复了，而这个节点原有数据是还在的，所以：推迟分片分配，从而减少ES的开销，具体做法如下：**
```json

PUT /_all/_settings
{
	"settings": {
		"index.unassigned.node_left.delayed_timeout": "5m"
	}
}

```
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105212840898-1177630417.png)

- 可以全局修改，也可以在建索引时修改



#### 4.16.3、带路由查询
- **前面说过：路由计算公式 `shard = hash( routing ) % number_of_primary_shards`**
- **而routing默认值就是文档id，所以查询时把文档id带上，如：前面玩kibana做的操作**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105213351198-1889428623.png)


- 不带路由就会把分片和副本都查出来，然后进行轮询，这效率想都想得到会慢一点嘛



#### 4.16.4、内存优化
- **修改es的config/jvm.options**
![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220105215239639-619799136.png)


- 把上面的数字改了，**Xms 表示堆的初始大小， Xmx 表示可分配的最大内存，ES默认是1G，这个数字在现实中是远远不够了，改它的目的是：为了能够在 Java 垃圾回收机制清理完堆内存后不需要重新分隔计算堆内存的大小而浪费资源，可以减轻伸缩堆大小带来的压力，但是也需要注意：改这两个数值，需要确保 Xmx 和 Xms 的大小是相同的，另外就是：这两个数值别操作32G啊，前面已经讲过了**







### 4.17、附上一些配置说明

| **参数名**                         | **参数值**    | **说明**                                                     |
| ---------------------------------- | ------------- | ------------------------------------------------------------ |
| cluster.name                       | elasticsearch | 配置 ES 的集群名称，默认值是 ES，建议改成与所存数据相关的名称， ES 会自动发现在同一网段下的 集群名称相同的节点 |
| node.name                          | node-1001     | 集群中的节点名，在同一个集群中不能重复。节点 的名称一旦设置，就不能再改变了。当然，也可以 设 置 成 服 务 器 的 主 机 名 称 ， 例 如 node.name: ${hostname} |
| node.master                        | true          | 指定该节点是否有资格被选举成为 Master 节点，默 认是 True，如果被设置为 True，则只是有资格成为 Master 节点，具体能否成为 Master 节点，需要通过选举产生 |
| node.data                          | true          | 指定该节点是否存储索引数据，默认为 True。数据的增、删、改、查都是在 Data 节点完成的 |
| index.number_of_shards             | 1             | 设置索引分片个数，默认是 1 片。也可以在创建索引时设置该值，具体设置为多大值要根据数据量的大小来定。如果数据量不大，则设置成 1 时效率最高 |
| index.number_of_replicas           | 1             | 设置默认的索引副本个数，默认为 1 个。副本数越多，集群的可用性越好，但是写索引时需要同步的数据越多 |
| transport.tcp.compress             | true          | 设置在节点间传输数据时是否压缩，默认为 False                 |
| discovery.zen.minimum_master_nodes | 1             | 设置在选举 Master 节点时需要参与的最少的候选主节点数，默认为 1。如果使用默认值，则当网络不稳定时有可能会出现脑裂。 合理的 数 值 为 ( master_eligible_nodes / 2 )+1 ， 其 中 master_eligible_nodes 表示集群中的候选主节点数 |
| discovery.zen.ping.timeout         | 3s            | 设置在集群中自动发现其他节点时 Ping 连接的超时时间，同时也是选主节点的延迟时间，默认为 3 秒。 在较差的网络环境下需要设置得大一点，防止因误判该节点的存活状态而导致分片的转移 |





### 4.19、说一些另外的理论吧
#### 4.19.1、ES的master主节点选举流程
- **1、首先选主是由ZenDiscovery来完成的（ 它做了两件事：一个是Ping过程 ———— 发现节点嘛 、二是Unicast过程 ———— 控制哪些节点需要Ping通 ）**

- **2、对所有可以成为master的节点（ 文件中设置的node.master: true ）根据nodeId字典排序，每次“选举节点（ 即：参与投票选举主节点的那个节点 ）”都把自己知道的节点排一次序，就是把排好序的第一个节点（ 第0位 ）认为是主节点（ 投一票 ）**

- **3、当某个节点的投票数达到一个值时（ （ 可以成为master节点数n / 2 ） + 1 ），而该节点也投自己，那么这个节点就是master节点，否则重新开始，直到选出master**

- **另外注意：master节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data节点可以关闭http功能**




#### 4.19.2、ES的集群脑裂问题
**导致的原因：**
- 网络问题：集群间的网络延迟导致一些节点访问不到master, 认为master 挂掉了从而选举出新的master,并对master上的分片和副本标红，分配新的主分片
- 节点负载：主节点的角色既为master又为data,访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点
- 内存回收：data 节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应


**脑裂问题解决方案：**

- **减少误判：discovery.zen ping_ timeout 节点状态的响应时间，默认为3s，可以适当调大，如果master在该响应时间的范围内没有做出响应应答，判断该节点已经挂掉了。调大参数（ 如6s，discovery.zen.ping_timeout:6 ），可适当减少误判**

- **选举触发：discovery.zen.minimum. _master_ nodes:1，该参數是用于控制选举行为发生的最小集群主节点数量。当备选主节点的个數大于等于该参数的值，且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为(n / 2) +1, n为主节点个数（即有资格成为主节点的节点个数）**

- **角色分离：即master节点与data节点分离，限制角色**

	- **主节点配置为：node master: true，node data: false**
	- **从节点置为：node master: false，node data: true**




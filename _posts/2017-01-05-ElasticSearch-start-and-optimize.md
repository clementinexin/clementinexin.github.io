---
layout: post
title:  "Elasticsearch入门级优化"
date: 2017-01-05
categories: Middleware,Elasticsearch
tags: Elasticsearch
published: true
---
* 目录
{:toc}


# 一、Elasticsearch 工具

## 1.监控插件之Head
在5.x上安装方式有所变化，不能直接在服务器上安装，只能在本机安装然后再链接上去

```plain
http://localhost:9100/
http://elastic:changeme@10.32.214.197:8410/
```

## 2.ElasticSearch监控之Xpack
安装Xpack之后自动启用BA认证，如果暂时不想认证，可以修改默认配置文件

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"


xpack.security.authc:
  anonymous:
    username: anonymous_user
    roles: superuser
    authz_exception: false
```

- [Kibana]

```plain
http://localhost:5601/login?next=%2Fapp%2Fkibana%23%2Fdev_tools%2Fconsole%3F_g%3D
```


# 二、Elasticsearch 索引与查询优化


## 1.without explain，默认不需要解释

### 查询带explain

```json
POST /hq/lowprice/_search
{
  "from" : 0,
  "size" : 500,
  "query" : {
    "bool" : {
      "must" : [
        {
          "terms" : {
            "org" : [
              "PEK",
              "NAY"
            ],
            "boost" : 1.0
          }
        },
        {
          "terms" : {
            "dst" : [
              "PVG",
              "TAO",
              "KMG",
              "CTU",
              "URC",
              "NAY",
              "HGH",
              "XIY",
              "CSX",
              "CGO",
              "WUH",
              "NKG",
              "SHA",
              "TSN",
              "SZX",
              "CAN",
              "SYX",
              "HAK",
              "PEK",
              "XMN",
              "CKG"
            ],
            "boost" : 1.0
          }
        }
      ],
      "disable_coord" : false,
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "post_filter" : {
    "bool" : {
      "must" : [
        {
          "range" : {
            "timestamp" : {
              "from" : 1483357435,
              "to" : null,
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        },
        {
          "range" : {
            "date" : {
              "from" : "2017-01-02",
              "to" : "2017-05-02",
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        },
        {
          "range" : {
            "discount" : {
              "from" : null,
              "to" : 70,
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        }
      ],
      "disable_coord" : false,
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "explain" : true,
  "sort" : [
    {
      "discount" : {
        "order" : "asc",
        "unmapped_type" : "long"
      }
    }
  ],
  "ext" : { }
}
```


查询结果

```json
"_explanation": {
          "value": 6.6199427,
          "description": "sum of:",
          "details": [
            {
              "value": 6.6199427,
              "description": "sum of:",
              "details": [
                {
                  "value": 2.7939816,
                  "description": "sum of:",
                  "details": [
                    {
                      "value": 2.7939816,
                      "description": "weight(org:PEK in 56957) [PerFieldSimilarity], result of:",
                      "details": [
                        {
                          "value": 2.7939816,
                          "description": "score(doc=56957,freq=1.0 = termFreq=1.0\n), product of:",
                          "details": [
                            {
                              "value": 2.7939816,
                              "description": "idf(docFreq=25412, docCount=415391)",
                              "details": []
                            },
                            {
                              "value": 1,
                              "description": "tfNorm, computed from:",
                              "details": [
                                {
                                  "value": 1,
                                  "description": "termFreq=1.0",
                                  "details": []
                                },
                                {
                                  "value": 1.2,
                                  "description": "parameter k1",
                                  "details": []
                                },
                                {
                                  "value": 0,
                                  "description": "parameter b (norms omitted for field)",
                                  "details": []
                                }
                              ]
                            }
                          ]
                        }
                      ]
                    }
                  ]
                },
                {
                  "value": 3.8259609,
                  "description": "sum of:",
                  "details": [
                    {
                      "value": 3.8259609,
                      "description": "weight(dst:NKG in 56957) [PerFieldSimilarity], result of:",
                      "details": [
                        {
                          "value": 3.8259609,
                          "description": "score(doc=56957,freq=1.0 = termFreq=1.0\n), product of:",
                          "details": [
                            {
                              "value": 3.8259609,
                              "description": "idf(docFreq=9054, docCount=415391)",
                              "details": []
                            },
                            {
                              "value": 1,
                              "description": "tfNorm, computed from:",
                              "details": [
                                {
                                  "value": 1,
                                  "description": "termFreq=1.0",
                                  "details": []
                                },
                                {
                                  "value": 1.2,
                                  "description": "parameter k1",
                                  "details": []
                                },
                                {
                                  "value": 0,
                                  "description": "parameter b (norms omitted for field)",
                                  "details": []
                                }
                              ]
                            }
                          ]
                        }
                      ]
                    }
                  ]
                }
              ]
            },
            {
              "value": 0,
              "description": "match on required clause, product of:",
              "details": [
                {
                  "value": 0,
                  "description": "# clause",
                  "details": []
                },
                {
                  "value": 0,
                  "description": "weight(_type:lowprice in 56957) [], result of:",
                  "details": [
                    {
                      "value": 0,
                      "description": "score(doc=56957,freq=1.0), with freq of:",
                      "details": [
                        {
                          "value": 1,
                          "description": "termFreq=1.0",
                          "details": []
                        }
                      ]
                    }
                  ]
                }
              ]
            }
          ]
        }
```

查询耗时

```json
{
  "took": 135,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 9940,
    "max_score": null,
    "hits": [
    ]
  }
}
```

设置explain false

```plain
  "explain" : false,
```

参考代码

```java
/**
 * Should each {@link org.elasticsearch.search.SearchHit} be returned with an
 * explanation of the hit (ranking).
 */
public SearchRequestBuilder setExplain(boolean explain) {
    sourceBuilder().explain(explain);
    return this;
}
```

### 禁用explain之后的结果

```json
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 10065,
    "max_score": null,
    "hits": [
    ]
  }
}
```

## 2.指定_source需要的字段

_source 指定DSL

```plain
  "_source": ["org","dst","date","discount","price" ],
```

查询结果耗时

```json
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 10973,
    "max_score": null,
    "hits": []
  }
}
```

参考代码

```java

    /**
     * Adds stored fields to load and return (note, it must be stored) as part of the search request.
     * To disable the stored fields entirely (source and metadata fields) use {@code storedField("_none_")}.
     * @deprecated Use {@link SearchRequestBuilder#storedFields(String...)} instead.
     */
    @Deprecated
    public SearchRequestBuilder fields(String... fields) {
        sourceBuilder().storedFields(Arrays.asList(fields));
        return this;
    }

    /**
     * Adds stored fields to load and return (note, it must be stored) as part of the search request.
     * To disable the stored fields entirely (source and metadata fields) use {@code storedField("_none_")}.
     */
    public SearchRequestBuilder storedFields(String... fields) {
        sourceBuilder().storedFields(Arrays.asList(fields));
        return this;
    }
```

## 3.post_filter vs query

### 指定先query再post_filter

```json
POST /hq/lowprice/_search
{
  "from" : 0,
  "size" : 500,
  "query" : {
    "bool" : {
      "must" : [
        {
          "terms" : {
            "org" : [
              "PEK",
              "NAY"
            ],
            "boost" : 1.0
          }
        },
        {
          "terms" : {
            "dst" : [
              "PVG",
              "TAO",
              "KMG",
              "CTU",
              "TNA",
              "URC",
              "NAY",
              "HGH",
              "XIY",
              "CSX",
              "CGO",
              "WUH",
              "NKG",
              "SHA",
              "TSN",
              "SZX",
              "CAN",
              "SYX",
              "HAK",
              "PEK",
              "XMN",
              "CKG"
            ],
            "boost" : 1.0
          }
        },
        {
          "range" : {
            "timestamp" : {
              "from" : 1483360715,
              "to" : null,
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        },
        {
          "range" : {
            "date" : {
              "from" : "2017-01-02",
              "to" : "2017-05-02",
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        },
        {
          "range" : {
            "discount" : {
              "from" : null,
              "to" : 70,
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        }
      ],
      "disable_coord" : false,
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "explain" : false,
  "sort" : [
    {
      "discount" : {
        "order" : "asc",
        "unmapped_type" : "long"
      }
    }
  ],
  "ext" : { }
}
```
参考代码，postFilter与query的区别

```java
/**
 * Constructs a new search source builder with a search query.
 *
 * @see org.elasticsearch.index.query.QueryBuilders
 */
public SearchRequestBuilder setQuery(QueryBuilder queryBuilder) {
    sourceBuilder().query(queryBuilder);
    return this;
}

/**
 * Sets a filter that will be executed after the query has been executed and only has affect on the search hits
 * (not aggregations). This filter is always executed as last filtering mechanism.
 */
public SearchRequestBuilder setPostFilter(QueryBuilder postFilter) {
    sourceBuilder().postFilter(postFilter);
    return this;
}
```

postFilter耗时


```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 9791,
    "max_score": null,
    "hits": []
  }
}
```

### 指定BoolQuery查询

```json
{
  "from" : 0,
  "size" : 500,
  "query" : {
    "bool" : {
      "must" : [
        {
          "terms" : {
            "org" : [
              "PEK",
              "NAY"
            ],
            "boost" : 1.0
          }
        },
        {
          "terms" : {
            "dst" : [
              "PVG",
              "TAO",
              "KMG",
              "CTU",
              "TNA",
              "URC",
              "NAY",
              "HGH",
              "XIY",
              "CSX",
              "CGO",
              "WUH",
              "NKG",
              "SHA",
              "TSN",
              "SZX",
              "CAN",
              "SYX",
              "HAK",
              "PEK",
              "XMN",
              "CKG"
            ],
            "boost" : 1.0
          }
        },
        {
          "bool" : {
            "must" : [
              {
                "range" : {
                  "timestamp" : {
                    "from" : 1483363289,
                    "to" : null,
                    "include_lower" : true,
                    "include_upper" : true,
                    "boost" : 1.0
                  }
                }
              },
              {
                "range" : {
                  "date" : {
                    "from" : "2017-01-02",
                    "to" : "2017-05-02",
                    "include_lower" : true,
                    "include_upper" : true,
                    "boost" : 1.0
                  }
                }
              },
              {
                "range" : {
                  "discount" : {
                    "from" : null,
                    "to" : 70,
                    "include_lower" : true,
                    "include_upper" : true,
                    "boost" : 1.0
                  }
                }
              }
            ],
            "disable_coord" : false,
            "adjust_pure_negative" : true,
            "boost" : 1.0
          }
        }
      ],
      "disable_coord" : false,
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "explain" : false,
  "sort" : [
    {
      "discount" : {
        "order" : "asc",
        "unmapped_type" : "long"
      }
    }
  ],
  "ext" : { }
}
```

> BoolQuery 比 PostFilter快的地方在于不需要取出所有查询符合的结果然后在内存中进行过滤，而是在查询的时候利用Filter缓存进行过滤，速度更快，占用内存更少


## 3.Scroll API，据说在查询大量符合条件的结果时比较有用，类似SQL的游标

[Using scrolls in Java](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-search-scrolling.html)

参考使用代码

```java
BoolQueryBuilder query = new BoolQueryBuilder();
if (!CollectionUtils.isEmpty(orgSet)) {
    query.must(QueryBuilders.termsQuery("org", orgSet));
}

if (!CollectionUtils.isEmpty(dstSet)) {
    query.must(QueryBuilders.termsQuery("dst", dstSet));
}

query.must(QueryBuilders.rangeQuery("timestamp").from(minTimestamp))
        .must(QueryBuilders.rangeQuery("date").from(startDate).to(endDate))
        .must(QueryBuilders.rangeQuery("discount").to(discountFilter));

QueryBuilder filter = QueryBuilders.boolQuery().must(QueryBuilders.rangeQuery("timestamp").from(minTimestamp))
        .must(QueryBuilders.rangeQuery("date").from(startDate).to(endDate))
        .must(QueryBuilders.rangeQuery("discount").to(discountFilter));
SortBuilder sort = SortBuilders.fieldSort(orderBy).order(SortOrder.ASC).unmappedType("long");

long start = System.currentTimeMillis();
SearchRequestBuilder builder = client.prepareSearch(index)
        .setTypes(type)
        .setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
        .setScroll(new TimeValue(60000))
        .setQuery(query).setSize(100)
        //.fields("org","dst","date","discount","price")
        //.setPostFilter(filter)
        .addSort(sort);

if(count != 0) {
    builder.setFrom(0).setSize(count);
}
builder.setExplain(false);

SearchResponse scrollResp = builder.get();

//Scroll until no hits are returned
do {
    for (SearchHit hit : scrollResp.getHits().getHits()) {
        //Handle the hit...
    }

    scrollResp = client.prepareSearchScroll(scrollResp.getScrollId()).setScroll(new TimeValue(60000)).execute().actionGet();
} while(scrollResp.getHits().getHits().length != 0); // Zero hits mark the end of the scroll and the while loop.

System.out.println("***** cost" + ( System.currentTimeMillis() - start));

System.out.printf(builder.toString());
SearchResponse response = builder.execute().actionGet();
System.out.println("\n----------- cost " + response.getTook() + " \n");
System.out.println(response.getHits().getTotalHits() + "\n");
```

> 响应时间在秒级，不提也罢

## 4.基于Lucene的底层查询

参考API，对lucene的掌握还是不够，暂时作罢

```java
/**
 * Converts this QueryBuilder to a lucene {@link Query}.
 * Returns <tt>null</tt> if this query should be ignored in the context of
 * parent queries.
 *
 * @param context additional information needed to construct the queries
 * @return the {@link Query} or <tt>null</tt> if this query should be ignored upstream
 */
Query toQuery(QueryShardContext context) throws IOException;

/**
 * Converts this QueryBuilder to an unscored lucene {@link Query} that acts as a filter.
 * Returns <tt>null</tt> if this query should be ignored in the context of
 * parent queries.
 *
 * @param context additional information needed to construct the queries
 * @return the {@link Query} or <tt>null</tt> if this query should be ignored upstream
 */
Query toFilter(QueryShardContext context) throws IOException;
```

## 指定SearchType

SearchType的枚举类型及其用法，Default就是QUERY_THEN_FETCH

```java
/**
 * Same as {@link #QUERY_THEN_FETCH}, except for an initial scatter phase which goes and computes the distributed
 * term frequencies for more accurate scoring.
 */
DFS_QUERY_THEN_FETCH((byte) 0),
/**
 * The query is executed against all shards, but only enough information is returned (not the document content).
 * The results are then sorted and ranked, and based on it, only the relevant shards are asked for the actual
 * document content. The return number of hits is exactly as specified in size, since they are the only ones that
 * are fetched. This is very handy when the index has a lot of shards (not replicas, shard id groups).
 */
QUERY_THEN_FETCH((byte) 1),
/**
 * Same as {@link #QUERY_AND_FETCH}, except for an initial scatter phase which goes and computes the distributed
 * term frequencies for more accurate scoring.
 */
DFS_QUERY_AND_FETCH((byte) 2),
/**
 * The most naive (and possibly fastest) implementation is to simply execute the query on all relevant shards
 * and return the results. Each shard returns size results. Since each shard already returns size hits, this
 * type actually returns size times number of shards results back to the caller.
 */
QUERY_AND_FETCH((byte) 3);
```

## 5.禁用source，用FiledData获取想要的字段

查看类型的映射，默认是带有source的

```plain
GET /realtime/quote/_mapping
```

```json
{
  "realtime": {
    "mappings": {
      "quote": {
        "properties": {
          "carrier": {
            "type": "keyword"
          },
          "date": {
            "type": "date"
          },
          "discount": {
            "type": "long"
          },
          "dst": {
            "type": "keyword"
          },
          "flightno": {
            "type": "keyword"
          },
          "org": {
            "type": "keyword"
          },
          "price": {
            "type": "long"
          },
          "quoteChannel": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "siteno": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "timestamp": {
            "type": "long"
          }
        }
      }
    }
  }
}
```

禁用source的DSL

```json
curl -XPUT 'localhost:9200/low?pretty' -d '{
    "mappings": {
      "quote": {
        "_source":{
          "enabled":false
        },
        "properties": {
          "carrier": {
            "type": "keyword"
          },
          "date": {
            "type": "date",
            "store": "yes"
          },
          "discount": {
            "type": "long"
          },
          "dst": {
            "type": "keyword"
          },
          "flightno": {
            "type": "keyword"
          },
          "org": {
            "type": "keyword"
          },
          "price": {
            "type": "long"
          },
          "quoteChannel": {
            "type": "keyword"
          },
          "siteno": {
            "type": "keyword"
          },
          "timestamp": {
            "type": "long"
          }
        }
      }
    }
  }'
{
  "acknowledged" : true,
  "shards_acknowledged" : true
}
```

插入测试数据

```json
curl -XPOST 'localhost:9200/low/quote?pretty' -d '{
          "org": "PEK",
          "dst": "NKG",
          "date": "2017-02-07",
          "discount": 10,
          "price": 200,
          "siteno": "715",
          "flightno": "MU2852",
          "carrier": "MU",
          "timestamp": 1483498141,
          "quoteChannel": "MT"
        }'
```

```json
curl -XPOST 'localhost:9200/hq/quote?pretty' -d '{
          "org": "PEK",
          "dst": "TAO",
          "date": "2017-01-31",
          "discount": 14,
          "price": 189,
          "siteno": "555",
          "flightno": "SC4660",
          "carrier": "SC",
          "timestamp": 1483508132,
          "quoteChannel": "MT"
        }'
```

用fileddata获取想要的字段

```json
curl -XGET 'localhost:9200/hq/quote/_search?pretty' -d '
{
  "from" : 0,
  "size" : 500,
  "_fields":["date","org","dst"],
  "query" : {
    "bool" : {
      "must" : [
        {
          "terms" : {
            "org" : [
              "PEK",
              "NAY"
            ],
            "boost" : 1.0
          }
        },
        {
          "terms" : {
            "dst" : [
              "PVG",
              "TAO",
              "KMG",
              "CTU",
              "TNA",
              "URC",
              "NAY",
              "HGH",
              "XIY",
              "CSX",
              "CGO",
              "WUH",
              "NKG",
              "SHA",
              "TSN",
              "SZX",
              "CAN",
              "SYX",
              "HAK",
              "PEK",
              "XMN",
              "CKG"
            ],
            "boost" : 1.0
          }
        },
        {
          "range" : {
            "date" : {
              "from" : "2017-01-02",
              "to" : "2017-05-02",
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        },
        {
          "range" : {
            "discount" : {
              "from" : null,
              "to" : 70,
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        }
      ],
      "disable_coord" : false,
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "explain" : false,
  "sort" : [
    {
      "discount" : {
        "order" : "asc",
        "unmapped_type" : "long"
      }
    }
  ],
  "ext" : { }
}'
```

测试数据禁用source

```json
curl -XPUT 'localhost:9200/corp?pretty' -d '{
    "mappings": {
      "emp": {
        "_source":{
          "enabled":false
        },
        "properties": {
        "name" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }'
```

插入测试数据

```json
curl -XPOST 'localhost:9200/corp/emp?pretty' -d '{
  "name":"John"
  }'
```

查询测试数据DSL

```json
curl -XGET 'localhost:9200/corp/emp/_search?pretty' -d '{
  "docvalue_fields": ["name"],
  "query" : {
    "bool" : {
      "must" : [
        {
          "term" : {
            "name" :
              "John"
          }
        }
      ]
    }
  }
}'
```

指定docvalue_fields的DSL

```json
curl -XGET 'localhost:9200/corp/emp/_search?pretty' -d '{
  "docvalue_fields": ["name"]
}'
```

查询结果

> Fielddata is disabled on text fields by default. Set fielddata=true on [name] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory.

```json
curl -XGET 'localhost:9200/corp/emp/_search?pretty' -d '{
  "docvalue_fields": ["name"]
}'
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 4,
    "failed" : 1,
    "failures" : [
      {
        "shard" : 1,
        "index" : "corp",
        "node" : "Hy4FJUuaSUWQiMy0Czsymg",
        "reason" : {
          "type" : "illegal_argument_exception",
          "reason" : "Fielddata is disabled on text fields by default. Set fielddata=true on [name] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory."
        }
      }
    ]
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [ ]
  }
}
```

设置fielddata的DSL

```json
curl -XPUT 'localhost:9200/corp/_mapping/emp' -d '
{
  "properties": {
    "name": {
      "type":     "text",
      "fielddata": true
    }
  }
}'
```

完整DSL

```json
curl -XGET 'localhost:9200/corp/_mapping?pretty'
{
  "corp" : {
    "mappings" : {
      "emp" : {
        "_source" : {
          "enabled" : false
        },
        "properties" : {
          "name" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            },
            "fielddata" : true
          }
        }
      }
    }
  }
}
```

- [_source](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)
- [字段数据](http://kibana.logstash.es/content/elasticsearch/performance/fielddata.html)
- [doc_values](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/doc-values.html)
- [fielddata](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/fielddata.html)



## 6.文档较少时更新主分片为1

更新索引的shards

```json
curl -XPUT '10.32.213.194:8410/hq' -d '{
   "settings" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 2
   }
}'

```

更新索引的副本shards

```json
curl -XPUT '10.32.213.194:8410/hq' -d '{
   "settings" : {
      "number_of_replicas" : 2
   }
}'

```

禁用source

```json
curl -XPUT '10.32.213.194:8410/hqquote/_mapping/hqlow' -d '{
  "hqlow":{
        "_source":{
          "enabled":false
        }
      }
}'

```

参考
- [Java Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-admin-indices.html)


Java API生成mapping，禁用source，指定shard数量

```java
private void createIndexNoSourceNoIndex(String index, String type){

    try {
        boolean exist = ESClient.INSTANCE.client.admin().indices().prepareExists(index).execute().actionGet().isExists();
        if (!exist) {
            XContentBuilder mapping = XContentFactory.jsonBuilder()
                    .startObject()
                    .startObject(type)
                    .startObject("_source").field("enabled",false).endObject()
                    .startObject("properties")
                    .startObject("org").field("type", "keyword").field("index", "not_analyzed").endObject()
                    .startObject("dst").field("type", "keyword").field("index", "not_analyzed").endObject()
                    .startObject("carrier").field("type", "keyword").field("index", "no").endObject()
                    .startObject("flightno").field("type", "keyword").field("index", "not_analyzed").endObject()
                    .startObject("date").field("type","date").endObject()
                    .startObject("discount").field("type","long").endObject()
                    .startObject("price").field("type","long").field("index","no").endObject()
                    .endObject()
                    .endObject()
                    .endObject();
            ESClient.INSTANCE.client.admin().indices().prepareCreate(index)
                    .setSettings(Settings.builder()
                            .put("index.number_of_shards", 1)
                            .put("index.number_of_replicas", 2)
                    )
                    .addMapping(type, mapping)
                    .execute().actionGet();
        }
    } catch (Exception e) {
        logger.error("{}", e);
    }
}
```


# 三、Elasticsearch 管理服务器

## 1.更改服务器shards

更新副本shards的DSL

```json
curl -XPUT '10.32.214.197:8410/hq/_settings' -d '{
    "index" : {
        "number_of_replicas" : 0
    }
}'
```

重启服务器时可能报如下错误，可设置副本shard为0，待集群健康状况恢复，再更新副本shard数量即可

```java
Caused by: org.elasticsearch.transport.RemoteTransportException: [node-3][10.32.213.195:8411][internal:index/shard/recovery/start_recovery]
Caused by: org.elasticsearch.index.engine.RecoveryEngineException: Phase[1] phase1 failed
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.recoverToTarget(RecoverySourceHandler.java:140) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.recover(PeerRecoverySourceService.java:119) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.access$100(PeerRecoverySourceService.java:54) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:128) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:125) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.transport.TransportRequestHandler.messageReceived(TransportRequestHandler.java:33) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.xpack.security.transport.SecurityServerTransportInterceptor$ProfileSecuredRequestHandler.messageReceived(SecurityServerTransportInterceptor.java:206) ~[?:?]
        at org.elasticsearch.transport.RequestHandlerRegistry.processMessageReceived(RequestHandlerRegistry.java:69) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.transport.TcpTransport$RequestHandler.doRun(TcpTransport.java:1348) ~[elasticsearch-5.0.2.jar:5.0.2]
        ... 5 more
Caused by: org.elasticsearch.indices.recovery.RecoverFilesRecoveryException: Failed to transfer [0] files with total size of [0b]
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.phase1(RecoverySourceHandler.java:335) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.recoverToTarget(RecoverySourceHandler.java:138) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.recover(PeerRecoverySourceService.java:119) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.access$100(PeerRecoverySourceService.java:54) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:128) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:125) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.phase1(RecoverySourceHandler.java:335) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.recoverToTarget(RecoverySourceHandler.java:138) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.recover(PeerRecoverySourceService.java:119) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.access$100(PeerRecoverySourceService.java:54) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:128) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:125) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.transport.TransportRequestHandler.messageReceived(TransportRequestHandler.java:33) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.xpack.security.transport.SecurityServerTransportInterceptor$ProfileSecuredRequestHandler.messageReceived(SecurityServerTransportInterceptor.java:206) ~[?:?]
        at org.elasticsearch.transport.RequestHandlerRegistry.processMessageReceived(RequestHandlerRegistry.java:69) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.transport.TcpTransport$RequestHandler.doRun(TcpTransport.java:1348) ~[elasticsearch-5.0.2.jar:5.0.2]
        ... 5 more
Caused by: java.lang.IllegalStateException: try to recover [hq][1] from primary shard with sync id but number of docs differ: 316679 (node-3, primary) vs 316715(node-1)
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.phase1(RecoverySourceHandler.java:224) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.RecoverySourceHandler.recoverToTarget(RecoverySourceHandler.java:138) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.recover(PeerRecoverySourceService.java:119) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService.access$100(PeerRecoverySourceService.java:54) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:128) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.indices.recovery.PeerRecoverySourceService$StartRecoveryTransportRequestHandler.messageReceived(PeerRecoverySourceService.java:125) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.transport.TransportRequestHandler.messageReceived(TransportRequestHandler.java:33) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.xpack.security.transport.SecurityServerTransportInterceptor$ProfileSecuredRequestHandler.messageReceived(SecurityServerTransportInterceptor.java:206) ~[?:?]
        at org.elasticsearch.transport.RequestHandlerRegistry.processMessageReceived(RequestHandlerRegistry.java:69) ~[elasticsearch-5.0.2.jar:5.0.2]
        at org.elasticsearch.transport.TcpTransport$RequestHandler.doRun(TcpTransport.java:1348) ~[elasticsearch-5.0.2.jar:5.0.2]
        ... 5 more
```

参考
[Github 解决办法](https://github.com/elastic/elasticsearch/issues/12661)
[indices-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html)



## 2.批量文档删除优化

有批量文档需要删除时，可尝试异步API  deletebyquery async

[java-docs-delete-by-query](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-docs-delete-by-query.html#CO2-4)



## 3.在线更新索引

现有索引
```json
{
  "flight": {
    "mappings": {
      "low": {
        "_source": {
          "enabled": false
        },
        "properties": {
          "carrier": {
            "type": "keyword"
          },
          "date": {
            "type": "date"
          },
          "discount": {
            "type": "long"
          },
          "dst": {
            "type": "keyword"
          },
          "flightno": {
            "type": "keyword"
          },
          "org": {
            "type": "keyword"
          },
          "price": {
            "type": "long"
          },
          "quoteChannel": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "siteno": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "timestamp": {
            "type": "long"
          }
        }
      }
    }
  }
}
```

```json
curl -XPUT '10.32.213.194:8410/hqquote_v1' -d '{
  "settings" : {
     "number_of_shards" : 1,
     "number_of_replicas" : 2
  },
    "mappings": {
      "low": {
        "_source": {
          "enabled": false
        },
        "properties": {
          "carrier": {
            "type": "keyword"
          },
          "date": {
            "type": "date"
          },
          "discount": {
            "type": "long"
          },
          "dst": {
            "type": "keyword"
          },
          "flightno": {
            "type": "keyword"
          },
          "org": {
            "type": "keyword"
          },
          "price": {
            "type": "long"
          },
          "quoteChannel": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "siteno": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "timestamp": {
            "type": "long"
          }
        }
      }
    }
}'

```

创建v2


为v1创建别名
```json
curl -XPOST '10.32.213.194:8410/_aliases' -d '
{
    "actions": [
        { "add": {
            "alias": "hqquote",
            "index": "hqquote_v1"
        }}
    ]
}
'
```

```json
curl -XPOST '10.32.213.194:8410/hqquote_v1/_alias/hqquote'
```

```json
curl -XPOST '10.32.213.194:8410/hqquote_v1/_alias/hqquote?pretty'

{
 "error" : {
   "root_cause" : [
     {
       "type" : "invalid_alias_name_exception",
       "reason" : "Invalid alias name [hqquote], an index exists with the same name as the alias",
       "index_uuid" : "GD1uwQ8hRoqNcsTIoztQng",
       "index" : "hqquote"
     }
   ],
   "type" : "invalid_alias_name_exception",
   "reason" : "Invalid alias name [hqquote], an index exists with the same name as the alias",
   "index_uuid" : "GD1uwQ8hRoqNcsTIoztQng",
   "index" : "hqquote"
 },
 "status" : 400
}
```

```json
curl -XPOST '10.32.213.195:8410/flight/_alias/flight_v1?pretty'

{
  "acknowledged" : true
}
```

```json
curl -XGET '10.32.213.194:8410/flight_v1/_alias/*?pretty'

{
  "flight" : {
    "aliases" : {
      "flight_v1" : { }
    }
  }
}
```

```json
curl -XGET '10.32.213.194:8410/flight_v1/?pretty'

{
  "flight" : {
    "aliases" : {
      "flight_v1" : { }
    },
    "mappings" : {
      "low" : {
        "_source" : {
          "enabled" : false
        },
        "properties" : {
          "carrier" : {
            "type" : "keyword"
          },
          "date" : {
            "type" : "date"
          },
          "discount" : {
            "type" : "long"
          },
          "dst" : {
            "type" : "keyword"
          },
          "flightno" : {
            "type" : "keyword"
          },
          "org" : {
            "type" : "keyword"
          },
          "price" : {
            "type" : "long"
          },
          "quoteChannel" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "siteno" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "timestamp" : {
            "type" : "long"
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1483588426866",
        "number_of_shards" : "5",
        "number_of_replicas" : "2",
        "uuid" : "d13JsUkVRFOLVJHcvjaZiw",
        "version" : {
          "created" : "5000299"
        },
        "provided_name" : "flight"
      }
    }
  }
}
```

hqquote _mapping

```json
{
  "hqquote": {
    "mappings": {
      "hqlow": {
        "properties": {
          "carrier": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "date": {
            "type": "date"
          },
          "discount": {
            "type": "long"
          },
          "dst": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "flightno": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "org": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "price": {
            "type": "long"
          },
          "quoteChannel": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "siteno": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "timestamp": {
            "type": "long"
          }
        }
      }
    }
  }
}
```

```json
curl -XPOST '10.32.213.194:8410/_reindex' -d '
{
  "source": {
    "index": "hqquote"
  },
  "dest": {
    "index": "hqquote_v1"
  }
}'
{"took":93802,
  "timed_out":false,
  "total":479600,
  "updated":0,
  "created":479600,
  "deleted":0,
  "batches":480,
  "version_conflicts":0,
  "noops":0,
  "retries":{"bulk":0,"search":0},
  "throttled_millis":0,
  "requests_per_second":-1.0,
  "throttled_until_millis":0,
  "failures":[]
  }%

```

```json
{
  "hqquote_v1": {
    "mappings": {
      "low": {
        "_source": {
          "enabled": false
        },
        "properties": {
          "carrier": {
            "type": "keyword"
          },
          "date": {
            "type": "date"
          },
          "discount": {
            "type": "long"
          },
          "dst": {
            "type": "keyword"
          },
          "flightno": {
            "type": "keyword"
          },
          "org": {
            "type": "keyword"
          },
          "price": {
            "type": "long"
          },
          "quoteChannel": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "siteno": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "timestamp": {
            "type": "long"
          }
        }
      }
    }
  }
}
```

# 四、Elasticsearch 监控方案


## plugin

- head,marvel,kopf
- xpack

## http API

- cluster/health
- cluster/_stats
- nodes/_stats
- [indices/_stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html)

## write a plugin

- [plugins example](https://github.com/elastic/elasticsearch/tree/master/plugins)

# 五、参考

- [深入理解elasticsearch](http://blog.zhenchuan.me/deep-into-elasticsearch/)
- [Elasticsearch 架构以及源码概览](http://jolestar.com/elasticsearch-architecture/)
- [ElasticSearch 中文文档](http://106.186.120.253/preview/index.html)
- [found-text-analysis-part-1](https://www.elastic.co/blog/found-text-analysis-part-1)
- [found-text-analysis-part-2](https://www.elastic.co/blog/found-text-analysis-part-2)
- [使用工具 Http ie](https://github.com/jkbrzt/httpie#json)

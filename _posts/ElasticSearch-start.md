




```plain
http://localhost:9100/
http://elastic:changeme@10.32.214.197:8410/
```


## ElasticSearch监控

- [Kibana]

```plain
http://localhost:5601/login?next=%2Fapp%2Fkibana%23%2Fdev_tools%2Fconsole%3F_g%3D
```

- [深入理解elasticsearch](http://blog.zhenchuan.me/deep-into-elasticsearch/)
- [Elasticsearch 架构以及源码概览](http://jolestar.com/elasticsearch-architecture/)



## without explain

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

```plain
  "explain" : false,
```

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

## 指定_source

```plain
  "_source": ["org","dst","date","discount","price" ],
```

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

## post_filter vs query

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

```json

```


## Scroll API

[Using scrolls in Java](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-search-scrolling.html)

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

响应时间在秒级，不提也罢

## 基于Lucene查询

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

## SearchType

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

## 禁用source

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

```json
curl -XPOST 'localhost:9200/corp/emp?pretty' -d '{
  "name":"John"
  }'
```

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

```json
curl -XGET 'localhost:9200/corp/emp/_search?pretty' -d '{
  "docvalue_fields": ["name"]
}'
```

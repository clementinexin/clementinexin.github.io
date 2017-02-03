---
layout: post
title:  "Elasticsearch 集群监控和安全"
date: 2017-01-13
categories: Elasticsearch
tags: Elasticsearch
published: true
---
* 目录
{:toc}

## Elasticsearch安全和维护


### 安装xpack插件

自从5.0之后Elasticsearch统一了原来的插件诸如shield、marvel之类的多个插件的功能，并且不再提供原来的安装方式，试用期是1个月，一个月之后便有如下的过期提示

```plain
#
# License [will expire] on [Friday, January 20, 2017]. If you have a new license, please update it.
# Otherwise, please reach out to your support contact.
#
# Commercial plugins operate with reduced functionality on license expiration:
# - security
#  - Cluster health, cluster stats and indices stats operations are blocked
#  - All data operations (read and write) continue to work
# - watcher
#  - PUT / GET watch APIs are disabled, DELETE watch API continues to work
#  - Watches execute and write to the history
#  - The actions of the watches don't execute
# - monitoring
#  - The agent will stop collecting cluster and indices metrics
#  - The agent will stop automatically cleaning indices older than [xpack.monitoring.history.duration]
# - graph
#  - Graph explore APIs are disabled

```

查询License有效期

```plain
curl -X GET -u elastic:changeme 'http://10.32.213.194:8410/_xpack/license'
{
  "license" : {
    "status" : "active",
    "uid" : "4cf1df87-5e18-47fe-9570-b9308ca71f26",
    "type" : "trial",
    "issue_date" : "2016-12-21T08:48:54.016Z",
    "issue_date_in_millis" : 1482310134016,
    "expiry_date" : "2017-01-20T08:48:54.016Z",
    "expiry_date_in_millis" : 1484902134016,
    "max_nodes" : 1000,
    "issued_to" : "specialoffer",
    "issuer" : "elasticsearch",
    "start_date_in_millis" : -1
  }
}
```

xpack过期之后的提示

```plain
#
# LICENSE [EXPIRED] ON [SUNDAY, JANUARY 08, 2017]. IF YOU HAVE A NEW LICENSE, PLEASE UPDATE IT.
# OTHERWISE, PLEASE REACH OUT TO YOUR SUPPORT CONTACT.
#
# COMMERCIAL PLUGINS OPERATING WITH REDUCED FUNCTIONALITY
# - security
#  - Cluster health, cluster stats and indices stats operations are blocked
#  - All data operations (read and write) continue to work
# - watcher
#  - PUT / GET watch APIs are disabled, DELETE watch API continues to work
#  - Watches execute and write to the history
#  - The actions of the watches don't execute
# - monitoring
#  - The agent will stop collecting cluster and indices metrics
#  - The agent will stop automatically cleaning indices older than [xpack.monitoring.history.duration]
# - graph
#  - Graph explore APIs are disabled
```


```plain
curl -X GET -u elastic:changeme 'http://localhost:9200/_xpack/license'
{
  "license" : {
    "status" : "expired",
    "uid" : "1417ffae-6f1f-4c5d-b4f6-c4c0fff068a8",
    "type" : "trial",
    "issue_date" : "2016-12-09T02:37:01.483Z",
    "issue_date_in_millis" : 1481251021483,
    "expiry_date" : "2017-01-08T02:37:01.483Z",
    "expiry_date_in_millis" : 1483843021483,
    "max_nodes" : 1000,
    "issued_to" : "elasticsearch",
    "issuer" : "elasticsearch",
    "start_date_in_millis" : -1
  }
}
```

### 启用认证，去掉匿名认证

```json
http :9200/customer/external/1
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="security" charset="UTF-8"
content-encoding: gzip
content-type: application/json; charset=UTF-8
transfer-encoding: chunked

{
    "error": {
        "header": {
            "WWW-Authenticate": "Basic realm=\"security\" charset=\"UTF-8\""
        },
        "reason": "missing authentication token for REST request [/customer/external/1]",
        "root_cause": [
            {
                "header": {
                    "WWW-Authenticate": "Basic realm=\"security\" charset=\"UTF-8\""
                },
                "reason": "missing authentication token for REST request [/customer/external/1]",
                "type": "security_exception"
            }
        ],
        "type": "security_exception"
    },
    "status": 401
}
```

### 卸载xpack插件
参考 [Uninstall x-pack](https://www.elastic.co/guide/en/x-pack/current/installing-xpack.html)

```plain
bin/elasticsearch-plugin remove x-pack
```

### 启用匿名认证

```plain
http.cors.enabled

Enable or disable cross-origin resource sharing, i.e. whether a browser on another origin can execute requests against Elasticsearch. Set to true to enable Elasticsearch to process pre-flight CORS requests. Elasticsearch will respond to those requests with the Access-Control-Allow-Origin header if the Origin sent in the request is permitted by the http.cors.allow-origin list. Set to false (the default) to make Elasticsearch ignore the Origin request header, effectively disabling CORS requests because Elasticsearch will never respond with the Access-Control-Allow-Origin response header. Note that if the client does not send a pre-flight request with an Origin header or it does not check the response headers from the server to validate the Access-Control-Allow-Origin response header, then cross-origin security is compromised. If CORS is not enabled on Elasticsearch, the only way for the client to know is to send a pre-flight request and realize the required response headers are missing.

http.cors.allow-origin

Which origins to allow. Defaults to no origins allowed. If you prepend and append a / to the value, this will be treated as a regular expression, allowing you to support HTTP and HTTPs. for example using /https?:\/\/localhost(:[0-9]+)?/ would return the request header appropriately in both cases. * is a valid value but is considered a security risk as your elasticsearch instance is open to cross origin requests from anywhere.
```

启用匿名访问

```yml
xpack.security.authc:
  anonymous:
    username: anonymous_user
    roles: superuser
    authz_exception: false
```

启用匿名认证之后，如果API中没有携带认证用户名和密码将提示如下

```java
Caused by: org.elasticsearch.ElasticsearchSecurityException: missing authentication token for action [cluster:monitor/nodes/liveness]

```

X-pack Java API

[java-clients](https://www.elastic.co/guide/en/x-pack/current/java-clients.html)

Elasticsearch启用匿名认证配置

```plain
http.cors.enabled: true
http.cors.allow-origin: "*"

xpack.security.enabled: true

xpack.security.authc:
  anonymous:
    username: client_user
    roles: transport_client
    authz_exception: true
```

### 更新默认密码

```plain
curl -XPUT localhost:9200/_xpack/security/user/elastic/_password -d '
{
  "password": "elasticpassword"
}'
```

## Elasticsearch迁移工具

```plain
elasticdump \
  --input=http://10.32.214.197:8410/hqquote \
  --output=http://localhost:9200/hqlowprice \
  --type=lowprice
```

> source禁用的index是无法dump的

```plain
./esm  -s http://10.32.214.197:8410   -d http://localhost:9200 -x hqquote  -w=10 -b=10 -c 10000 --sliced_scroll_size=10
[01-13 11:15:28] [INF] [main.go:387,main] start data migration..
Scroll 10068190 / 10068190 [======================================================================================================================================================================================] 100.00% 17m29s
Bulk 10067924 / 10068190 [========================================================================================================================================================================================] 100.00% 22m54s
[01-13 11:38:23] [INF] [main.go:410,main] data migration finished.
```

## 基于Nginx的Elasticsearch基本认证


## 基于Nginx彻底Elasticsearch多角色授权

- 没有认证的用户只能使用ping命令，ping指定的URL(/)；
- 认证过的user用户可以执行_search与_analyze请求；
- 认证过的admin用户可以执行任何请求。

我们使用不同的端口为不同的角色服务：

```plain
events {
  worker_connections  1024;
}

http {

  upstream elasticsearch {
      server 127.0.0.1:9200;
  }

  # Allow HEAD / for all
  #
  server {
      listen 8080;

      location / {
        return 401;
      }

      location = / {
        if ($request_method !~ "HEAD") {
          return 403;
          break;
        }

        proxy_pass http://elasticsearch;
        proxy_redirect off;
      }
  }

  # Allow access to /_search and /_analyze for authenticated "users"
  #
  server {
      listen 8081;

      auth_basic           "Elasticsearch Users";
      auth_basic_user_file users;

      location / {
        return 403;
      }

      location ~* ^(/_search|/_analyze) {
        proxy_pass http://elasticsearch;
        proxy_redirect off;
      }
  }

  # Allow access to anything for authenticated "admins"
  #
  server {
      listen 8082;

      auth_basic           "Elasticsearch Admins";
      auth_basic_user_file admins;

      location / {
        proxy_pass http://elasticsearch;
        proxy_redirect off;
      }
  }

}

```

## 参考

[elasticsearch-migration](https://github.com/medcl/elasticsearch-migration)
[elasticsearch-dump](https://github.com/taskrabbit/elasticsearch-dump)

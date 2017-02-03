---
layout: post
title:  "Nginx入手指南"
date: 2017-01-20
categories: Server
tags: nginx
published: true
---
* 目录
{:toc}

## Nginx上手

安装结果说明了文档的根路径、默认端口、配置文件位置、启动和停止指令等。

```plain
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
==> Summary
🍺  /usr/local/Cellar/nginx/1.10.2_1: 8 files, 979.8K
```

```plain
brew services  start nginx
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)
```

>第一次启动并没有报错，但是实际访问8080端口时，页面显示HelloWorld 显然不是Nginx的默认页面，而且左上角的favion是一个SpringSource的标志，应该是某个Java应用启动了


lsof -i tcp:8080查看端口占用的进程

```plain
lsof -i tcp:8080
COMMAND  PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME


WeChat   330   dj   98u  IPv6 0x1f3a86e7f7b5813b      0t0  TCP 172.30.14.169:56033->140.207.119.142:http-alt (ESTABLISHED)
java    9915   dj  221u  IPv6 0x1f3a86e7db7b067b      0t0  TCP *:http-alt (LISTEN)
```

kill该应用之后，8080页面还是无法访问，于是重启Nginx

```plain
brew services stop nginx
Stopping `nginx`... (might take a while)
```

至此显示Nginx首页了

## Nginx配置文件详解

```plain
# 运行用户
user www-data;
# 启动进程,通常设置成和cpu的数量相等
worker_processes  1;

# 全局错误日志及PID文件
error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;

# 工作模式及连接数上限
events {
    use epoll; #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
    worker_connections 1024; #单个后台worker process进程的最大并发链接数
    # multi_accept on;
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    #设定mime类型,类型由mime.type文件定义
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    access_log    /var/log/nginx/access.log;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    #将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞
    tcp_nopush      on;
    tcp_nodelay     on;
    #连接超时时间
    keepalive_timeout  65;

    #开启gzip压缩
    gzip  on;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";

    #设定请求缓冲
    client_header_buffer_size    1k;
    large_client_header_buffers  4 4k;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

    #设定负载均衡的服务器列表
    upstream mysvr {
        #weigth参数表示权值，权值越高被分配到的几率越大
        #本机上的Squid开启3128端口
        server 192.168.8.1:3128 weight=5;
        server 192.168.8.2:80  weight=1;
        server 192.168.8.3:80  weight=6;
    }


    server {
        #侦听80端口
        listen       80;
        #定义使用www.xx.com访问
        server_name  www.xx.com;

        #设定本虚拟主机的访问日志
        access_log  logs/www.xx.com.access.log  main;

        #默认请求
        location / {
            root   /root;      #定义服务器的默认网站根目录位置
            index index.php index.html index.htm;   #定义首页索引文件的名称

            fastcgi_pass  www.xx.com;
            fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
            include /etc/nginx/fastcgi_params;
        }

        # 定义错误提示页面
        error_page   500 502 503 504 /50x.html;
            location = /50x.html {
            root   /root;
        }

        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            root /var/www/virtual/htdocs;
            #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }
        #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ \.php$ {
            root /root;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /home/www/www$fastcgi_script_name;
            include fastcgi_params;
        }
        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status            on;
            access_log              on;
            auth_basic              "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }
        #禁止访问 .htxxx 文件
        location ~ /\.ht {
            deny all;
        }

    }

    #第一个虚拟服务器
    server {
        #侦听192.168.8.x的80端口
        listen       80;
        server_name  192.168.8.x;

        #对aspx后缀的进行负载均衡请求
        location ~ .*\.aspx$ {
            root   /root;#定义服务器的默认网站根目录位置
            index index.php index.html index.htm;#定义首页索引文件的名称

            proxy_pass  http://mysvr;#请求转向mysvr 定义的服务器列表

            #以下是一些反向代理的配置可删除.
            proxy_redirect off;

            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size 10m;    #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k;  #缓冲区代理缓冲用户端请求的最大字节数，
            proxy_connect_timeout 90;  #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90;        #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90;         #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k;             #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
            proxy_busy_buffers_size 64k;    #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k;  #设定缓存文件夹大小，大于这个值，将从upstream服务器传
        }
    }
}
```

### 配置说明

- main 全局配置
- events events模块来用指定nginx的工作模式和工作模式及连接数上限
- http http模块可以说是最核心的模块了，它负责HTTP服务器相关属性的配置，它里面的server和upstream子模块，至关重要，等到反向代理和负载均衡以及虚拟目录等会仔细说。
- server sever 模块是http的子模块，它用来定一个虚拟主机
- location location 根据它字面意思就知道是来定位的，定位URL，解析URL，所以，它也提供了强大的正则匹配功能，也支持条件判断匹配，用户可以通过location指令实现Nginx对动、静态网页进行过滤处理。

```plain
# user字段表明了Nginx服务是由哪个用户哪个群组来负责维护进程的，默认是nobody
# 我这里用了cainengtian用户，staff组来启动并维护进程
# 查看当前用户命令： whoami
# 查看当前用户所属组命令： groups ，当前用户可能有多个所属组，选第一个即可
user cainengtian staff;

# worker_processes字段表示Nginx服务占用的内核数量
# 为了充分利用服务器性能你可以直接写你本机最高内核
# 查看本机最高内核数量命令： sysctl -n hw.ncpu
worker_processes 4;

# error_log字段表示Nginx错误日志记录的位置
# 模式选择：debug/info/notice/warn/error/crit
# 上面模式从左到右记录的信息从最详细到最少
error_log  /usr/local/var/logs/nginx/error.log debug;

# Nginx执行的进程id,默认配置文件是注释了
# 如果上面worker_processes的数量大于1那Nginx就会启动多个进程
# 而发信号的时候需要知道要向哪个进程发信息，不同进程有不同的pid，所以写进文件发信号比较简单
# 你只需要手动创建，比如我下面的位置： touch /usr/local/var/run/nginx.pid
pid  /usr/local/var/run/nginx.pid;

events {
    # 每一个worker进程能并发处理的最大连接数
    # 当作为反向代理服务器，计算公式为： `worker_processes * worker_connections / 4`
    # 当作为HTTP服务器时，公式是除以2
    worker_connections  2048;
}

http {
    # 关闭错误页面的nginx版本数字，提高安全性
    server_tokens off;
    include       mime.types;
    default_type  application/octet-stream;

    # 日志记录格式，如果关闭了access_log可以注释掉这段
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                 '$status $body_bytes_sent "$http_referer" '
    #                '"$http_user_agent" "$http_x_forwarded_for"';

    # 关闭access_log可以让读取磁盘IO操作更快
    # 当然如果你在学习的过程中可以打开方便查看Nginx的访问日志
    access_log off;

    sendfile        on;

    # 在一个数据包里发送所有头文件，而不是一个接一个的发送
    tcp_nopush     on;

    # 不要缓存
    tcp_nodelay on;

    keepalive_timeout  65;

    gzip  on;
    client_max_body_size 10m;
    client_body_buffer_size 128k;

    # 关于下面这段在后面紧接着来谈！
    include /usr/local/etc/nginx/sites-enabled/*;
}
```

server配置分离，把不同站点的配置放在不同的配置文件中，只需要在http模块中include就行

```plain
server {
    # Nginx监听端口号
    listen       80;
    # 服务器的名字，默认为localhost，你也可以写成aotu.jd.com，这样子就可以通过aotu.jd.com来访问
    server_name  localhost;
    # 代码放置的根目录
    root /var/www/;
    # 编码
    charset utf-8;

    location / {
        # index字段声明了解析的后缀名的先后顺序
        # 下面匹配到/的时候默认找后缀名为php的文件，找不到再找html，再找不到就找htm
        index index.php index.html index.htm;
        # 自动索引
        autoindex on;
        # 这里引入了解析PHP的东西
        include /usr/local/etc/nginx/conf.d/php-fpm;
    }

    # 404页面跳转到404.html，相对于上面的root目录
    error_page  404              /404.html;
    # 403页面跳转到403.html，相对于上面的root目录
    error_page  403              /403.html;
    # 50x页面跳转到50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

## Nginx相关指令

检查配置文件语法和重新加载配置文件

```plain
nginx -t && nginx -s reload
```

启动停止和重启

```plain
brew services start nginx
brew services stop nginx
brew services restart nginx
```

## Nginx配置反向代理

- 正向代理
> 正向代理也就是代理，他的工作原理就像一个跳板，简单的说，我访问不了google.com，但是我能访问一个代理服务器A，A能访问google.com，于是我先连上代理服务器A，告诉他我需要google.com的内容，A就去取回来，然后返回给我。从网站的角度，只在代理服务器来取内容的时候有一次记录，有时候并不知道是用户的请求，也隐藏了用户的资料，这取决于代理告不告诉网站。

结论就是，正向代理是一个位于客户端和原始服务器(origin server)之间的服务器。为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

- 反向代理
> 举个例子，比如我想访问 http://www.test.com/readme，但www.test.com上并不存在readme页面，于是他是偷偷从另外一台服务器上取回来，然后作为自己的内容返回用户，但用户并不知情。这里所提到的 www.test.com 这个域名对应的服务器就设置了反向代理功能。

结论就是，反向代理正好相反，对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间(name-space)中的内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端，就像这些内容原本就是它自己的一样。


```plain
server {
    listen 80;
    server_name aotu.jd.com;
    root /var/www/;
    location /o2blog_wx/ {
        # 反向代理我们通过proxy_pass字段来设置
        # 也就是当访问http://aotu.jd.com/o2blog_wx的时候经过Nginx反向代理到服务器上的http://127.0.0.1:3000
        # 同时由于解析到服务器上的时候o2blog_wx这个字段都要处理
        # 所以通过rewrite字段来进行正则匹配替换
        # 也就是http://aotu.jd.com/o2blog_wx/hello经过Nginx解析到服务器变成http://127.0.0.1:3000/hello
        proxy_pass http://127.0.0.1:3000;
        rewrite ^/o2blog_wx/(.*) /$1 break;
    }
}
```

## 基于域名的虚拟主机

域名配置

```plain
127.0.0.1 www.iyangyi.com iyangyi.com
127.0.0.1 api.iyangyi.com
127.0.0.1 admin.iyangyi.com
```

```plain
# www.iyangyi.conf
server {
    listen 80;
    server_name www.iyangyi.com iyangyi.com;
    root /Users/yangyi/www/www.iyangyi.com/;
    index index.php index.html index.htm;
    access_log /usr/local/var/log/nginx/www.iyangyi.access.log main;
    error_log /usr/local/var/log/nginx/www.iyangyi.error.log error;
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        include        fastcgi.conf;
    }
}
# api.iyangyi.conf
server {
    listen 80;
    server_name api.iyangyi.com;
    root /Users/yangyi/www/api.iyangyi.com/;
    index index.php index.html index.htm;
    access_log /usr/local/var/log/nginx/api.iyangyi.access.log main;
    error_log /usr/local/var/log/nginx/api.iyangyi.error.log error;
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        include        fastcgi.conf;
    }
}
# admin.iyangyi.conf
server {
    listen 80;
    server_name admin.iyangyi.com;
    root /Users/yangyi/www/admin.iyangyi.com/;
    index index.php index.html index.htm;
    access_log /usr/local/var/log/nginx/admin.iyangyi.access.log main;
    error_log /usr/local/var/log/nginx/admin.iyangyi.error.log error;
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        include        fastcgi.conf;
    }
}

```

## Nginx配置限制访问

```plain
# 当匹配到/info的时候只允许10.7.101.224访问，其它的全部限制
# 同时改写为/info.php
location = /info {
    allow 10.7.101.224;
    deny all;
    rewrite (.*) /info.php
}
```

## Nginx配置负载均衡

Nginx的负载均衡模块目前支持4种调度算法:

- 1.weight 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。weight。指定轮询权值，weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。
- 2.ip_hash。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。
- 3.fair。比上面两个更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。
- 4.url_hash。按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包。

在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：

- down，表示当前的server暂时不参与负载均衡。
- backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
- max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
- fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。
注意 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。

### 基于weight

```plain
upstream webservers{
    server 192.168.33.11 weight=10;
    server 192.168.33.12 weight=10;
    server 192.168.33.13 weight=10;
}
server {
    listen 80;
    server_name upstream.iyangyi.com;
    access_log /usr/local/var/log/nginx/upstream.iyangyi.access.log main;
    error_log /usr/local/var/log/nginx/upstream.iyangyi.error.log error;
    location / {
        proxy_pass http://webservers;
        proxy_set_header  X-Real-IP  $remote_addr;
    }
}

upstream webservers{
    server 192.168.33.11 weight=10 max_fails=2 fail_timeout=30s;
    server 192.168.33.12 weight=10 max_fails=2 fail_timeout=30s;
    server 192.168.33.13 weight=10 max_fails=2 fail_timeout=30s;
}
max_fails : 允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。

fail_timeout : 在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用，进行健康状态检查。


upstream webservers{
    server 192.168.33.11 down;
    server 192.168.33.12 weight=10 max_fails=2 fail_timeout=30s;
    server 192.168.33.13 backup;
}
down 表示这台机器暂时不参与负载均衡。相当于注释掉了。

backup 表示这台机器是备用机器，是其他的机器不能用的时候，这台机器才会被使用

```

### 基于IP hash


```plain
upstream webservers{
    ip_hash;
    server 192.168.33.11 weight=1 max_fails=2 fail_timeout=30s;
    server 192.168.33.12 weight=1 max_fails=2 fail_timeout=30s;
    server 192.168.33.13 down;
}

ip_hash 模式下，最好不要设置weight参数，因为你设置了，就相当于手动设置了，将会导致很多的流量分配不均匀。

ip_hash模式下, backup参数不可用，加了会报错，为啥呢？因为，本身我们的访问就是固定的了，其实，备用已经不管什么作用了。
```


## Nginx配置页面缓存

页面缓存也是日常web 开发中很重要的一个环节，对于一些页面，我们可以将其静态化，保存起来，下次请求时候，直接走缓存，而不用去请求反相代理服务器甚至数据库服务了。从而减轻服务器压力。

nginx 也提供了简单而强大的下重定向，反向代理的缓存功能，只需要简单配置下，就能将指定的一个页面缓存起来。它的原理也很简单，就是匹配当前访问的url, hash加密后，去指定的缓存目录找，看有没有，有的话就说明匹配到缓存了。

```plain
http {
  proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=cache_zone:10m inactive=1d max_size=100m;
  upstream myproject {
    .....
  }
  server  {
    ....
    location ~* \.php$ {
        proxy_cache cache_zone; #keys_zone的名字
        proxy_cache_key $host$uri$is_args$args; #缓存规则
        proxy_cache_valid any 1d;
        proxy_pass http://127.0.0.1:8080;
    }
  }
  ....
}
```
1. path是用来指定 缓存在磁盘的路径地址。比如：/data/nginx/cache。那以后生存的缓存文件就会存在这个目录下。

2. levels用来指定缓存文件夹的级数，可以是：levels=1, levels=1:1, levels=1:2, levels=1:2:3 可以使用任意的1位或2位数字作为目录结构分割符，如 X, X:X,或 X:X:X 例如: 2, 2:2, 1:1:2，但是最多只能是三级目录。

那这个里面的数字是什么意思呢。表示取hash值的个数。比如：现在根据请求地址localhost/index.php?a=4 用md5进行哈希，得到e0bd86606797639426a92306b1b98ad9

levels=1:2 表示建立2级目录，把hash最后1位(9)拿出建一个目录，然后再把9前面的2位(ad)拿来建一个目录, 那么缓存文件的路径就是/data/nginx/cache/9/d/e0bd86606797639426a92306b1b98ad9

以此类推：levels=1:1:2表示建立3级目录，把hash最后1位(9)拿出建一个目录，然后再把9前面的1位(d)建一个目录, 最后把d前面的2位(8a)拿出来建一个目录 那么缓存文件的路径就是/data/nginx/cache/9/d/8a/e0bd86606797639426a92306b1b98ad9

3. keys_zone 所有活动的key和元数据存储在共享的内存池中，这个区域用keys_zone参数指定。one指的是共享池的名称，10m指的是共享池的大小。

注意每一个定义的内存池必须是不重复的路径，例如：

proxy_cache_path  /data/nginx/cache/one  levels=1      keys_zone=one:10m;
proxy_cache_path  /data/nginx/cache/two  levels=2:2    keys_zone=two:100m;
proxy_cache_path  /data/nginx/cache/three  levels=1:1:2  keys_zone=three:1000m;
4. inactive 表示指定的时间内缓存的数据没有被请求则被删除，默认inactive为10分钟。inactive=1d 1小时。inactive=30m30分钟。

5. max_size 表示单个文件最大不超过的大小。它被用来删除不活动的缓存和控制缓存大小，当目前缓存的值超出max_size指定的值之后，超过其大小后最少使用数据（LRU替换算法）将被删除。max_size=10g表示当缓存池超过10g就会清除不常用的缓存文件。

6. clean_time 表示每间隔自动清除的时间。clean_time=1m 1分钟清除一次缓存

proxy_cache 用来指定用哪个keys_zone的名字，也就是用哪个目录下的缓存。上面我们指定了三个one, two,three 。比如，我现在想用one 这个缓存目录 : proxy_cache one

proxy_cache_key 这个其实蛮重要的，它用来指定生成hash的url地址的格式。他会根据这个key映射成一个hash值，然后存入到本地文件。
proxy_cache_key $host$uri表示无论后面跟的什么参数，都会访问一个文件，不会再生成新的文件。
而如果proxy_cache_key $is_args$args，那么传入的参数 localhost/index.php?a=4 与localhost/index.php?a=44将映射成两个不同hash值的文件。

proxy_cache_key 默认是 "$scheme$host$request_uri"。但是一般我们会把它设置成：$host$uri$is_args$args 一个完整的url路径。

proxy_cache_valid 它是用来为不同的http响应状态码设置不同的缓存时间,

proxy_cache_valid  200 302  10m;
proxy_cache_valid  404      1m;
表示为http status code 为200和302的设置缓存时间为10分钟，404代码缓存1分钟。
如果只定义时间：

proxy_cache_valid 5m;
那么只对代码为200, 301和302的code进行缓存。
同样可以使用any参数任何相响应：

proxy_cache_valid  200 302 10m;
proxy_cache_valid  301 1h;
proxy_cache_valid  any 1m; #所有的状态都缓存1小时

```plain
proxy_cache_path /usr/local/var/cache levels=1:2 keys_zone=cache_zone:10m inactive=1d max_size=100m;
server  {
    listen 80;
    server_name cache.iyangyi.com;
    access_log /usr/local/var/log/nginx/cache.iyangyi.access.log main;
    error_log /usr/local/var/log/nginx/cache.iyangyi.error.log error;
    add_header X-Via $server_addr;
    add_header X-Cache $upstream_cache_status;
    location / {
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_cache cache_zone;
        proxy_cache_key $host$uri$is_args$args;
        proxy_cache_valid 200 304 1m;
        proxy_pass http://192.168.33.11;
    }
}
```

当然缓存文件夹 /usr/local/var/cache得提前新建好。然后重启nginx。

192.168.33.11 是apache服务器，在index.html页面就写了一个web1。

我们打开浏览器访问 cache.iyangyi.com 。就能看到web1了。

打开审核元素或者firebug。看network网络请求选项，我们可以看到，Response Headers，在这里我们可以看到：

X-Cache:MISS
X-Via:127.0.0.1
X-cache 为 MISS 表示未命中，请求被传送到后端。y因为是第一次访问，没有缓存，所以肯定是未命中。我们再刷新下，就发现其变成了HIT, 表示命中。它还有其他几种状态：

MISS 未命中，请求被传送到后端
HIT 缓存命中
EXPIRED 缓存已经过期请求被传送到后端
UPDATING 正在更新缓存，将使用旧的应答
STALE 后端将得到过期的应答
BYPASS 缓存被绕过了
我们再去看看缓存文件夹 /usr/local/var/cache里面是否有了文件：

cache git:(master) cd a/13
➜  13 git:(master) ls
5bd1af99bcb0db45c8bd601d9ee9e13a
➜  13 git:(master) pwd
/usr/local/var/cache/a/13
已经生成了缓存文件。

我们在url 后面随便加一个什么参数，看会不会新生成一个缓存文件夹及文件： http://cache.iyangyi.com/?w=ww55。因为我们使用的生成规则是全部url转换(proxy_cache_key $host$uri$is_args$args;)

查看 X-cache 为 MISS，再刷新 ，变成HIT。再去看一下缓存文件夹 /usr/local/var/cache。

  ~cache git:(master) ls
  4 a
果然又生成了一个4文件夹。

## 趟坑

### 配置基本认证

```plain
events {
  worker_connections  1024;
}

http {

  upstream elasticsearch {
    server 127.0.0.1:9200;
  }

  server {
    listen 8080;

    auth_basic "Protected Elasticsearch";
    auth_basic_user_file passwords;

    location / {
      proxy_pass http://elasticsearch;
      proxy_redirect off;
    }
  }

}
```

生成密码
$ printf "john:$(openssl passwd -crypt s3cr3t)\n" > passwords

不带密码访问

```html
curl -i "127.0.0.1:8080"
HTTP/1.1 401 Unauthorized
Server: nginx/1.10.2
Date: Thu, 19 Jan 2017 11:24:37 GMT
Content-Type: text/html
Content-Length: 195
Connection: keep-alive
WWW-Authenticate: Basic realm="Protected Elasticsearch"

<html>
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.10.2</center>
</body>
</html>
```

```html
curl -i "sp:s3cr3t@127.0.0.1:8080"
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Thu, 19 Jan 2017 11:24:32 GMT
Content-Type: application/json; charset=UTF-8
Content-Length: 327
Connection: keep-alive

{
  "name" : "NRZcmnC",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "dQeW6t-oQAiKVeKovVZaMA",
  "version" : {
    "number" : "5.0.2",
    "build_hash" : "f6b4951",
    "build_date" : "2016-11-24T10:07:18.101Z",
    "build_snapshot" : false,
    "lucene_version" : "6.2.1"
  },
  "tagline" : "You Know, for Search"
}
```

**注意** 密码文件的默认根目录是nginx目录，如果要把密码文件放在子目录中，要在server中指定子目录，否则读取不到密码文件报错403，而不是401

```html
curl -i "sp:s3cr3t@127.0.0.1:8080"
HTTP/1.1 403 Forbidden
Server: nginx/1.10.2
Date: Thu, 19 Jan 2017 11:17:09 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive

<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.10.2</center>
</body>
</html>
```


## 参考

- [学习手册：Web 开发环境与服务器生产环境](https://ninghao.net/blog/2136)
- [Nginx如何处理一个请求](http://tengine.taobao.org/nginx_docs/cn/docs/http/request_processing.html)
- [使用nginx的http验证功能对elasticsearch加密](http://www.jianshu.com/p/7ec26c13abbb)
- [nginx的配置、虚拟主机、负载均衡和反向代理（3](https://www.zybuluo.com/phper/note/133244)

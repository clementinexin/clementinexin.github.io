---
layout: post
title:  "Hystrix"
date: 2017-01-03
categories: Java,Middleware
tags: Java,Middleware,Hystrix
published: true
---
* 目录
{:toc}


**what**
- 是什么

**why**
- 为什么

## gradle引入依赖

```properties
#spring boot
theSpringBootVersion=1.3.3.RELEASE
theSpringCloudVersion=Brixton.RC1
theSpringPlatformBomVersion=2.0.3.RELEASE
theDependencyManagementPlugin=0.6.1.RELEASE
```

```gradle
dependencies {
    compile("org.springframework.boot:spring-boot-starter")
    compile("org.springframework.boot:spring-boot-starter-actuator")

    compile 'org.springframework.cloud:spring-cloud-starter-eureka'
    compile 'org.springframework.cloud:spring-cloud-starter-hystrix'
    compile 'org.springframework.cloud:spring-cloud-starter-hystrix-dashboard'
}
```

## Hystrix监控

监控Http API

```plain
http://localhost:9090/quote_rt/health
```

```json
{
  "status": "UP",
  "discoveryComposite": {
    "description": "Discovery Client not initialized",
    "status": "UNKNOWN",
    "discoveryClient": {
      "description": "Discovery Client not initialized",
      "status": "UNKNOWN"
    }
  },
  "diskSpace": {
    "status": "UP",
    "total": 249779191808,
    "free": 25734070272,
    "threshold": 10485760
  },
  "refreshScope": {
    "status": "UP"
  },
  "hystrix": {
    "status": "UP"
  }
}
```

## 效果

![](/{{site.url}}/assets/2017/01/Hystrix-turbine.png)

![](/{{site.url}}/assets/2017/02/Hystrix-dashboard.png)


## 参考

### SpringBoot整合

- [The Netflix stack, using Spring Boot - Part 1: Eureka](https://blog.de-swaef.eu/the-netflix-stack-using-spring-boot/)
- [The Netflix stack, using Spring Boot - Part 2: Hystrix](https://blog.de-swaef.eu/the-netflix-stack-using-spring-boot-part-2-hystrix/)
- [The Netflix stack, using Spring Boot - Part 3: Feign](https://blog.de-swaef.eu/the-netflix-stack-using-spring-boot-part-3-feign/)

### Hystrix监控

- [Hystrix](https://eacdy.gitbooks.io/spring-cloud-book/content/2%20Spring%20Cloud/2.4.1%20Hystrix.html)
- [Hystrix Dashboard](https://eacdy.gitbooks.io/spring-cloud-book/content/2%20Spring%20Cloud/2.4.2%20Hystrix%20Dashboard.html)
- [Turbine](https://eacdy.gitbooks.io/spring-cloud-book/content/2%20Spring%20Cloud/2.4.3%20Turbine.html)

---
layout: post
title:  "SpringBoot and Swagger2"
date: 2017-01-01
categories: Java,Middleware
tags: Java,SpringBoot,Swagger2,Middleware
published: true
---
* 目录
{:toc}

## 引入依赖

### for Gradle

```gradle
compile "io.springfox:springfox-swagger2:2.6.1"
compile 'io.springfox:springfox-swagger-ui:2.6.1'
```

### for Maven

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
</dependency>
```

## SpringMvc 配置

```xml

<mvc:default-servlet-handler/>


指明RequestHandler参考xsd

<xsd:element name="default-servlet-handler">
   <xsd:annotation>
      <xsd:documentation
         source="java:org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler"><![CDATA[
Configures a handler for serving static resources by forwarding to the Servlet container's default Servlet.  Use of this
handler allows using a "/" mapping with the DispatcherServlet while still utilizing the Servlet container to serve static
resources.
This handler will forward all requests to the default Servlet. Therefore it is important that it remains last in the
order of all other URL HandlerMappings. That will be the case if you use the "annotation-driven" element or alternatively
if you are setting up your customized HandlerMapping instance be sure to set its "order" property to a value lower than
that of the DefaultServletHttpRequestHandler, which is Integer.MAX_VALUE.
      ]]></xsd:documentation>
   </xsd:annotation>
   <xsd:complexType>
      <xsd:attribute name="default-servlet-name" type="xsd:string">
         <xsd:annotation>
            <xsd:documentation><![CDATA[
The name of the default Servlet to forward to for static resource requests.  The handler will try to auto-detect the container's
default Servlet at startup time using a list of known names.  If the default Servlet cannot be detected because of using an unknown
container or because it has been manually configured, the servlet name must be set explicitly.
            ]]></xsd:documentation>
         </xsd:annotation>
      </xsd:attribute>
   </xsd:complexType>
</xsd:element>
```

## 启用Swagger2

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * Created by daijiajia on 2016/12/27.
 */
@Configuration
@EnableSwagger2
@EnableWebMvc
@ComponentScan(value = "com.xxx.controller")
public class Swagger2Config extends WebMvcConfigurationSupport {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.xxx.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("特价机票接口文档")
                .description("特价机票接口文档")
                .termsOfServiceUrl("")
                .contact(new Contact("xxx", "MT", "xxx@xxx.com"))
                .version("1.0")
                .build();
    }
}
```

## 添加接口说明

```java
@ApiOperation(value = "平台异地接口", notes = "请求参数中的城市名已经URDEncode")
@ApiImplicitParams(
        {
                @ApiImplicitParam(name = "orgCityName", value = "出发城市", required = true, dataType = "String"),
                @ApiImplicitParam(name = "dstCityName", value = "到达城市", required = true, dataType = "String"),
                @ApiImplicitParam(name = "startDate", value = "开始日期yyyy-MM-dd", required = true, dataType = "String"),
                @ApiImplicitParam(name = "endDate", value = "结束日期yyyy-MM-dd", required = true, dataType = "String"),
                @ApiImplicitParam(name = "appPlatform", value = "应用平台,android还是ios", required = true, dataType = "String"),
                @ApiImplicitParam(name = "appVersion", value = "应用版本", required = true, dataType = "String"),
                @ApiImplicitParam(name = "appName", value = "应用名称,mt还是dp", required = true, dataType = "String")
        }
)
@RequestMapping(value = "", method = RequestMethod.GET)
@ResponseBody
```

## 效果

![]({{site.url}}}}/assets/2017/01/Swagger2.png)

## 参考
- [springfox](https://springfox.github.io/springfox/docs/current/)

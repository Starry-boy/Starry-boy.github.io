---
title: "Zuul Gateway 入门"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
  - SpringCloud
  - Zuul GetWay
  - 网关
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - SpringCloud
  - Zuul GetWay
  - 网关
description: 
---
# Zuul Gateway 入门

> 有点像 nginx 或者 servlet过滤器, 对外隐藏微服务的域名和端口,只需要知道 微服务名字和 uri 即刻访问指定 微服务，方便统一管理

## Pom

```xml
<dependencies>
        <!--Zuul 网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>

        <!--Eureka客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
```

## Main

```java
/**
 * @author ratel
 * @date 2020/4/5
 */
@EnableZuulProxy //标记此应用为 Zuul Gateway
@EnableDiscoveryClient
@EnableEurekaClient
@SpringBootApplication
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class,args);
    }
}
```

## Config

```yml
spring:
  application:
    name: @artifactId@
server:
  port: 8070
#  servlet:
#    context-path: /zuul

# eureka 注册中心配置
eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka1.ratel.com:8001/eureka/

# 指定路由规则
zuul:
  routes:
#    api-a:
#      path: /order/** #匹配路径
#      serviceId: ratel-microservice-order #微服务名字
#      url: http://localhost:8082 #重定向，不依赖 eureka
#简写
    ratel-microservice-order: #微服务名字
      path: /order/** #匹配规则
#  ignored-services: ratel-microservice-order #忽略掉这个微服务，否则url以 /ratel-microservice-order 为前缀的也会访问 order 服务
logging:
  level:
    root: debug

```

## 进阶配置

### 请求转发

```yml
# 指定路由规则
# 匹配优先级 按配置先后循序
zuul:
  routes:
    api-foward: #微服务名字 或 url 前缀
      path: forward:/test #表示将匹配上的 请求转发到当前网关的接口上
```





## Example

- 不使用网关

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405230940650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

- 使用Zuul网关

  [外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-XQGine4l-1586100144062)(/Users/ratel/Library/Application Support/typora-user-images/image-20200405231532598.png)]



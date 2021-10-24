---
title: "SpringCloud HystrixDashboard入门"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
  - HystrixDashboard
  - SpringCloud
  - 监控
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - HystrixDashboard
  - SpringCloud
  - 监控
description: 
---
# SpringCloud HystrixDashboard入门

## pom

```xml
    <dependencies>
        <!--springboot actuator 监控和管理Spring Boot应用-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--hystrix服务熔断-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--hystrix 仪表盘-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

## main

```java
/**
 * @author ratel
 * @date 2020/3/28
 */

@SpringCloudApplication
@EnableHystrixDashboard
public class HystrixDashboardApplication {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class,args);
    }
}
```

## config

```yml
server:
  port: 8050
  servlet:
    context-path: /dashboard

eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    service-url:
      deafultZone: http://eureka1.ratel.com:8001/eureka/
```

## 监控页面

> 启动项目 访问 hystrix 监控页面 http://localhost:8050/dashboard/hystrix

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040515443381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

输入被监控的应用的URL

**example**  http://localhost:8022/order/actuator/hystrix.stream

### 注意

> 首次访问 界面会显示 reading 状态, 这是因为 被监控的应用没有 被请求, 随便访问下接口 界面就会发生变化了，如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405160806370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

## Turbine 监控集群应用

**需要配合 hystrixDashboard 一起使用**

### Pom

```xml
<dependencies>
        <!--springboot actuator 监控和管理Spring Boot应用-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--burbine hystrix dashboard 集群应用监控中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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

### Main

```java
/**
 * @author ratel
 * @date 2020/4/5
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableTurbine
public class TurbineApplication {
    public static void main(String[] args) {
        SpringApplication.run(TurbineApplication.class,args);
    }
}

```

### Config

```yml
server:
  port: 8060
  servlet:
    context-path: /turbine

eureka:
  client:
    service-url:
      defaultZone: http://eureka1.ratel.com:8001/eureka/
  instance:
    prefer-ip-address: true

spring:
  application:
    name: @artifactId@

turbine:
  app-config: RATEL-MICROSERVICE-ORDER #指定监控的服务名
  cluster-name-expression: new String('default') #指定集群名字 默认集群名为 default
  combine-host-port: true #按服务名加端口 区分为不同的应用
  instanceUrlSuffix: order/actuator/hystrix.stream #如果被监控的应用带有前缀，则需要在这里配上后缀
```

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-2HDcDujl-1586082785157)(/Users/ratel/Library/Application Support/typora-user-images/image-20200405183124452.png)]

### 注意

> 访问 解控页面 可能会一直显示 loading 状态，只有被 @HystrixCommand 修饰的方法 才会被监控，所以随便访问个带有 @HystrixCommand 修饰的接口即可

## Hystrix Dashboard 整合 MQ

待续。。。

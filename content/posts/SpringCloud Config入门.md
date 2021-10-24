---
title: "SpringCloud Bus 入门"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
  - 消息总线
  - SpringCloud
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - SpringCloud
  - 消息总线
description: 
---
# SpringCloud Bus 入门

## 前言

> bus 分布式消息总线，目的是为了 弥补 Spring Config 的不足：当微服务数量过多时，每次修改配置文件 都需要调用 每个微服务的刷新端点 去刷新配置，非常麻烦，Bus 通过 消息中间件 订阅主题的方式，实现修改配置文件的同时，广播刷新所有订阅主题的微服务

## 注意

> Bus 支持的消息中间件 只有 RabbiMQ 和 Kafka

## 安装RabbitMQ

> https://blog.csdn.net/weixin_43723635/article/details/105621500

## Config Server 配置

### Pom

> 现有配置上追加 BabbitMQ 和 actautor

```xml
<dependencies>
        <dependency>
            <groupId>com.ratel</groupId>
            <artifactId>ratel-common-base</artifactId>
            <scope>compile</scope>
        </dependency>
        <!--SpringBoot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!--eurake客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--SpringCloud Config-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <!--springboot actuator 监控和管理Spring Boot应用-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--SpringCloud-Bus-RabbitMQ支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <!--SpringCloud-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
```

### Config 

> bootstrap 现有配置上追加

```yml
server:
  port: 8080

spring:
  application:
    name: @artifactId@
  cloud:
    config:
      server:
        git:
          uri: "https://github.com/jxc19960306/ratel-config.git" #仓库地址
#          username: "" #用户名
#          password: "" #密码

  rabbitmq: # Bus RabbitMQ 支持
    host: localhost #地址
    port: 5672 #端口
    username: ratel #用户名
    password: ratel #密码
  #      uri: http://localhost:8080 # ConfigServer url

eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone:  http://eureka1.ratel.com:8001/eureka/
    register-with-eureka: true
```

### Main

```java
/**
 * @author ratel
 * @date 2020/4/6
 */
@EnableConfigServer
@SpringBootApplication
@EnableEurekaClient
public class ConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class,args);
    }
}
```

## Config Client

### Pom

```xml
<dependencies>
        <dependency>
            <groupId>com.ratel</groupId>
            <artifactId>ratel-common-base</artifactId>
        </dependency>
        <!--SpringCloud config 客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!--springboot actuator 监控和管理Spring Boot应用-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--Eureka客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--Ribbon 客户端负载均衡-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        <!--SpringCloud-Bus-RabbitMQ支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <!--swagger2 api测试框架-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>
```

### Config

- bootstrap.yml

  ```yml
  spring:
    cloud:
      config:
        label: master #分支
        profile: dev #环境 example ratel-dev
        name: ratel #配置文件名 example ratel-dev
        discovery:
          enabled: true
          service-id: RATEL-CONFIG
  
    rabbitmq: # Bus RabbitMQ 支持
      host: localhost #地址
      port: 5672 #端口
      username: ratel #用户名
      password: ratel #密码
  #      uri: http://localhost:8080 # ConfigServer url
  
  eureka:
    instance:
      prefer-ip-address: true
    client:
      register-with-eureka: true
      service-url:
        defaultZone: http://eureka1.ratel.com:8001/eureka/
  ```

- application.yml

  ```yml
  server:
    port: 8022
    servlet:
      context-path: /order
  
  spring:
    application:
      name: @artifactId@
  
  management:
    endpoints:
      web:
        exposure:
          include: '*' #springboot2.0之后需要加上这个才会开放所有端点
  
    security:
      enabled: false #关闭端口访问授权控制
    endpoint:
      health:
        show-details: always #访问 health端点时获取详细信息 PS:自定义 healthIndicator 必须开启次选项
  
  #访问info端点时会显示下面的配置信息
  info:
    app: ${spring.application.name}
    version: @project.version@
  
  
  logging:
    level:
      root: debug
  feign:
    compression: #开启gzip 数据压缩
      request:
        enabled: true  #请求数据压缩
        mime-types: application/json #指定需要压缩的数据类型,全部压缩则去掉该选项
        min-request-size: 2048 #指定数据什么时候开始压缩
      response:
        enabled: true #响应数据压缩
    hystrix:
      enabled: true
  ```

### Main

```java
/**
 * @author ratel
 * @date 2020/3/22
 */
@EnableFeignClients("com.ratel.microservice")
@ComponentScan("com.ratel.microservice")
@SpringCloudApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }
}
```

### Controller

```java
/**
 * @author ratel
 * @date 2020/4/12
 */
@RestController
@RequestMapping("config")
@RefreshScope
public class ConfigController {
    @Value("${message}")
    private String message;

    @GetMapping("test")
    public RatelResponse test(){
        return new RatelResponse().ok(message);
    }
}

```

## Example

### 

1. 启动 Config Server

2. 访问 Rabbit Web

   Rabbit中增加一个主题

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419234348677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

3. 启动 Config Client

4. 修改 github 仓库中的配置文件

   - https://github.com/jxc19960306/ratel-config/blob/master/ratel-dev.yml

     ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419235523544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419235738796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

5. 访问 Config Server 查看 配置

   配置生效了

   [外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-tbmUYw4E-1587312913131)(/Users/ratel/Library/Application Support/typora-user-images/image-20200419235844798.png)]

6. 访问 Config Client 查看配置

   配置未生效，需要 访问 Config Server 的 refresh 端点，来刷新配置

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200420000156464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

7. 访问 Config Server 的 refresh 端点，来刷新配置

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200420000447911.png)

8. 再次访问 Config Client 查看配置是否刷新

   配置生效

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042000130837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)



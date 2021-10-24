---
title: "SpringCloud Feign入门"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
  - Feign
  - SpringCloud
  - RPC
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - Feign
  - SpringCloud
  - RPC
description: 
---
# SpringCloud Feign入门

## 依赖

```xml
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
     <!--Feign客户端-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>
		 <!--hystrix 服务降级时要用到-->
     <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
          <version>${spring-cloud-hystrix.version}</version>
      </dependency>
```

## 服务提供者

```java
/**
 * @author ratel
 * @date 2020/3/22
 */
@Slf4j
@RestController
public class SearchFeignClient implements SearchFeignApi {
    @Autowired
    private SearchService searchService;

    @Override
    public RatelResponse<Map<String, String>> get(Long id) {
        Map<String, String> map = searchService.get(id);
        return new RatelResponse<Map<String, String>>().ok(map);
    }
}
```

## 服务消费者

```java
/**
 * @author ratel
 * @date 2020/3/22
 */
@Slf4j
@RestController
@RequestMapping("order")
public class OrderController {
    @Autowired
    private OrderService orderService;
    @Autowired
    private SearchFeignApi searchFeignApi;

    @GetMapping("get/{id}")
    public RatelResponse<Map<String,String>> get(@PathVariable("id") Long id){
        return new RatelResponse<Map<String,String>>().ok(orderService.get(id));
    }

    @GetMapping("/search/get/{id}")
    public RatelResponse<Map<String,String>> searchGet(@PathVariable("id") Long id){
        return searchFeignApi.get(id);
    }
}

```

## Feign接口

> 自动义一个 接口,标注注解 @FeignClient 表示这是一个Feign 的客户端

注意：不要在接口类上 @RequestMapping 注解，

			不要在接口类上 @RequestMapping 注解，
	
			不要在接口类上 @RequestMapping 注解,否则启动项目时 会报错提示有 同名的 url，或者访问接口时方法执行了 			却报错404

```java
@FeignClient(name = "ratel-microservice-search",path = "/search",fallback = SearchFeignApiFallBack.class)
public interface SearchFeignApi {

    @PostMapping("feignApi/search/get/{id}")
    RatelResponse<Map<String,String>> get(@PathVariable("id") Long id);
}
```

`@FeginClient参数`

| 参数            | 含义                                                         | 是否必填 | 使用场景                                         |
| --------------- | ------------------------------------------------------------ | -------- | ------------------------------------------------ |
| value           | 微服务名 （spring.application.name）                         | 必填     | 从Eureka中获取指定服务                           |
| serviceId       |                                                              |          |                                                  |
| contextId       |                                                              |          |                                                  |
| name            | 和 value 是同一个字段                                        |          |                                                  |
| qualifier       |                                                              |          |                                                  |
| url             | 手动指定 接口调用地址l                                       |          | 测试                                             |
| decode404       | 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException |          |                                                  |
| configuration   | feign配置类                                                  |          | 可以指定Encoder,Decoder,LogLevel,Contract        |
| fallback        | 降级方法                                                     |          | 远程调用失败或超时的时候，会调用指定方法返回数据 |
| fallbackFactory | 如名和上面一直                                               |          | 指定通用的降级方法                               |
| path            | context 前缀                                                 |          | 如果访问的服务 带有前缀，则要加上这个，否则404   |
| primary         |                                                              |          |                                                  |

## Feign 服务降级演示

配置文件

```yml
feign:
  compression: #开启gzip 数据压缩
    request:
      enabled: true  #请求数据压缩
      mime-types: application/json #指定需要压缩的数据类型,全部压缩则去掉该选项
      min-request-size: 2048 #指定数据什么时候开始压缩
    response:
      enabled: true #响应数据压缩
  hystrix: 
    enabled: true #增加hystrix支持
```

主启动类

```java
/**
 * @author ratel
 * @date 2020/3/22
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients("com.ratel.microservice") //注意如果 feign接口在其他模块，则需要手动指定包扫描
@ComponentScan("com.ratel.microservice")
@EnableHystrix //启动hystrix支持
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }
}
```

指定 fallback方法

```java
@Component
@RequestMapping("/fallback/search")
public class SearchFeignApiFallBack implements SearchFeignApi {
    @Override
    public RatelResponse<Map<String, String>> get(Long id) {
        return new RatelResponse<>().err(ResponseCodeEnum.FailED,"调用失败，快速响应");
    }

    public SearchFeignApiFallBack() {
        System.out.println("SearchFeignApiFallBack init");
    }
}
```

修改服务提供者方法，睡眠一段时间 导致消费者调用超时，从而调用 fallback方法

```java
/**
 * @author ratel
 * @date 2020/3/22
 */
@Slf4j
@RestController
public class SearchFeignClient implements SearchFeignApi {
    @Autowired
    private SearchService searchService;

    @Override
    public RatelResponse<Map<String, String>> get(Long id) {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Map<String, String> map = searchService.get(id);
        return new RatelResponse<Map<String, String>>().ok(map);
    }
}
```

fallback调用结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020032423190815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

## Feign 服务降级演示 fallbackFactory

- 修改 SearchFeignApi

  ```java
  //去掉fallback 改成fallbackFactory 指定异常时调用的工厂
  @FeignClient(name = "ratel-microservice-search",path = "/search",fallbackFactory = FeignErrorLogFallBackFactory.class)
  public interface SearchFeignApi {
  
      @PostMapping("feignApi/search/get/{id}")
      RatelResponse<Map<String,String>> get(@PathVariable("id") Long id);
  }
  ```

- 增加一个类 FeignErrorLogFallBackFactory

  ```java
  /**
   * @author ratel
   * @date 2020/3/25
   */
  @Slf4j
  @Component
  public class FeignErrorLogFallBackFactory implements FallbackFactory<SearchFeignApi> {
      @Override
      public SearchFeignApi create(Throwable throwable) {
          log.error("feign client invoke rpc exception! error message ====>>>>{}",throwable.getMessage());
          return new SearchFeignApi() {
                  @Override
                  public RatelResponse<Map<String, String>> get(Long id) {
                      return new RatelResponse<>().err(ResponseCodeEnum.FAILED,"调用失败，feignFallFactory 快速响应");
                  }
          } ;
      }
  
      public FeignErrorLogFallBackFactory() {
          log.info(this.getClass().getName() + "   init !!!!");
      }
  }
  ```

- 测试结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200325165431671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

- 小结

  > 使用fallbackFactory 和 使用fallback 差别不大，fallbackFacktory 可以获取到异常信息

## 服务降级优化

> 使用fallback 每个feign的客户端有要单独配置一个降级的处理类,过于麻烦，可以偷个懒简化一下



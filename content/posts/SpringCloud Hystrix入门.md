---
title: "SpringCloud Hystrix入门"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
  - Hystrix
  - SpringCloud
  - 服务熔断、降级
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - HystrixDashboard
  - SpringCloud
  - 监控
description: 
---
# Hystrix

## Pom

```xml
	  <!--hystrix服务熔断-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

## application.yml

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
    enabled: true #启用hystrix
```

## main

```java
/**
 * @author ratel
 * @date 2020/3/22
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients("com.ratel.microservice")
@ComponentScan("com.ratel.microservice")
//@EnableDiscoveryClient
//@SpringBootApplication
//@EnableHystrix //启用 hystrix
@SpringCloudApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }
}
```

## 注意

>@EnableDiscoveryClient
>@SpringBootApplication
>@EnableHystrix  ==  @EnableCircuitBreaker
>这三个注解可以用一个注解替代  @SpringCloudApplication
>@SpringCloudApplication 是一个复合注解，上面包含了这三个注解

## 测试Hystrix服务降级

### 消费者

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
    @HystrixCommand(fallbackMethod = "fallback") //指定rpc调用异常 使用的降级方法名称
    public RatelResponse searchGet(@PathVariable("id") Long id){
        return searchFeignApi.get(id);
    }

  	//降级方法 方法入参 和 返回值 必须和 ↑↑↑↑ 的方法一致
    private RatelResponse fallback( Long id){
        return new RatelResponse().err(ResponseCodeEnum.FAILED,"rpc 调用失败，使用 hystrix fallback");
    }
}
```

### 提供者

```java
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

### 接口

```java
@FeignClient(name = "ratel-microservice-search",path = "/search")
public interface SearchFeignApi {

    @PostMapping("feignApi/search/get/{id}")
    RatelResponse get(@PathVariable("id") Long id);

}

```

### 调用结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200326105640260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

## 测试Hystrix缓存

> 添加@CacheResult注解,修改提供者 让他每次放返回数据不一样

### 消费者

```java
    @GetMapping("/search/get/{id}")
    @HystrixCommand(fallbackMethod = "fallback",ignoreExceptions = NullPointerException.class)
    @CacheResult  //添加@CacheResult注解
    public RatelResponse searchGet(@PathVariable("id") Long id){
        return searchFeignApi.get(id);
    }
```

### 提供者

```java
  @Override
    public RatelResponse<Map<String, String>> get(Long id) {
        Map<String, String> map = new HashMap<>();
        map.put(new Random(10).nextInt()+"","");
        return new RatelResponse<Map<String, String>>().ok(map);
    }
```

### 调用结果

<font color='red'>测试失败,回头再找问题</font>

## 测试Hystrix请求合并

### 消费者

```java
    @GetMapping("/search/get/{id}")
    public RatelResponse searchGet( @PathVariable("id") Long id){
        List<RatelResponse> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            String str = searchFeignApi.getStr(id);
            System.out.println(str);
        }
        return new RatelResponse().ok(list);
    }
```

### 提供者

```java
@Slf4j
@RestController
public class SearchFeignClient implements SearchFeignApi {
    @Autowired
    private SearchService searchService;

    @Override
    public RatelResponse<Map<String, String>> get(Long id) {
        Map<String, String> map = new HashMap<>();
        map.put(new Random().nextInt(10)+"","");
        log.info(JSON.toJSONString(map));
        return new RatelResponse<Map<String, String>>().ok(map);
    }

    @Override
    public String getStr(Long id) {
        log.info("单个getStr方法");
        return id+"";
    }

    @Override
    public List<String> batchGetStr(List<Long> ids) {
        log.info("批量getStr方法");
        return Arrays.asList("1","2");
    }
}
```

### 接口

```java
@FeignClient(name = "ratel-microservice-search",path = "/search")
public interface SearchFeignApi {

    @PostMapping("feignApi/search/get/{id}")
    @HystrixProperty(name = "requestCache.enabled",value = "true")
    @CacheResult
    RatelResponse get(@CacheKey @PathVariable("id") Long id);

    @PostMapping("feignApi/search/getStr/{id}")
    String getStr(@PathVariable("id")Long id);

    @HystrixCommand
    @PostMapping("feignApi/search/batchGetStr")
    List<String> batchGetStr(@RequestBody List<Long> ids);
}

```

### 调用结果

<font color='red'>测试失败 裂开</font>

## 注解配置

### @HystrixCommand

| 字段                    | 含义               | 使用场景                                                     |
| ----------------------- | ------------------ | ------------------------------------------------------------ |
| fallbackMethod          | 指定服务降级方法名 | rpc调用时发生异常,会调用指定fallback方法                     |
| groupKey                | 组名               | 往下翻                                                       |
| commandKey              | 方法名             | 往下翻                                                       |
| threadPoolKey           | 线程池名           | 往下翻                                                       |
| commandProperties       |                    |                                                              |
| threadPoolProperties    |                    |                                                              |
| ignoreExceptions        | 忽略指定异常       | 例如不希望空指针时调用降级方法 ignoreExceptions=NullPointerException.class |
| observableExecutionMode |                    |                                                              |
| raiseHystrixExceptions  |                    |                                                              |

### @HystrixProperty

### @HystrixCollapser

## Hystrix属性

Hystrix配置可以分为四个级别，优先级从低到高分别为：全局默认配置、全局配置、实例默认值、实例配置。

#### execution

`execution.isolation.strategy`： 该属性用来设置执行的隔离策略，它有如下两个选项。

1. `THREAD`: 通过线程池隔离的策略。它在独立的线程上执行， 并且它的并发限制受线程池中线程数量的限制。
2. `SEMAPHORE`: 通过信号量隔离的策略。它在调用线程上执行， 并且它的并发限制受信号量计数的限制。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | THREAD                                                       |
| 全局配置     | hystrix.command.default.execution.isolation.strategy         |
| 实例默认值   | @HystrixProperty(name=”execution.isolation.strategy”, value=”THREAD”) |
| 实例配置     | hystrix.command.HystrixCommandKey.execution.isolation.strategy |

实例配置中的HystrixCommandKey对应@HystrixCommand注解中commandKey 属性指定的值。

`execution.isolation.thread.timeoutinMilliseconds`： 该属性用来配置HystrixCommand执行的超时时间，单位为毫秒。当HystrixCommand执行时间超过该配置值之后， Hystrix会将该执行命令标记为TIMEOUT并进入服务降级处理逻辑。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 1000亳秒                                                     |
| 全局配置     | hystrix.command.default.execution.isolation.thread. timeoutinMilliseconds |
| 实例默认值   | @HystrixProperty(name=”execution.isolation.thread.timeoutinMilliseconds”,value=”2000”) |
| 实例配置     | hystrix.command.HystrixCommandKey.execution.isolation.thread.timeoutinMilliseconds |

`execution.timeout.enabled`: 该属性用来配置HystrixCommand的执行是否启用超时时间。默认为true, 如果设置为false, 那么`execution.isolation.thread.timeoutinMilliseconds`属性的配置将不再起作用。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.command.default.execution.timeout.enabled            |
| 实例默认值   | @HystrixProperty(name=”execution.timeout.enabled”, value=”false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.execution.timeout.enabled  |

`execution.isolation.thread.interruptOnTimeout`: 该属性用来配置当HystrixCommand执行超时的时候是否要将它中断。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.command.default.execution.isolation.thread.interruptOnTimeout |
| 实例默认值   | @HystrixProperty(name=”execution.isolation.thread.interruptOnTimeout”,value=”false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.execution.isolation.thread.interruptOnTimeout |

`execution.isolation.thread.interruptOnCancel`: 该属性用来配置当HystrixCommand执行被取消的时候是否要将它中断。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.command.default.execution.isolation.thread.interruptOnCancel |
| 实例默认值   | @HystrixProperty(name=”execution.isolation.thread.interruptOnCancel”,value= “false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.execution.isolation.thread.interruptOnCancel |

`execution.isolation.semaphore.maxConcurrentRequests`: 当HystrixCommand的隔离策略使用信号量的时候，该属性用来配置信号量的大小（并发请求数）。当最大并发请求数达到该设置值时，后续的请求将会被拒绝。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 10                                                           |
| 全局配置     | hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests |
| 实例默认值   | @HystrixProperty(name=”execution.isolation.semaphore.maxConcurrentRequests”, value=”2”) |
| 实例配置     | hystrix.command.HystrixCommandKey.execution.isolation.semaphore.maxConcurrentRequests |

#### fallback

`fallback.enabled`: 该属性用来设置服务降级策略是否启用，如果设置为false,那么当请求失败或者拒绝发生时，将不会调用降级服务。

| 属性级别     | 默认值、配置方式、配置属性                                |
| :----------- | :-------------------------------------------------------- |
| 全局默认配置 | true                                                      |
| 全局配置     | hystrix.command.default.fallback.enabled                  |
| 实例默认值   | @HystrixProperty(name= “fallback.enabled”, value=”false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.fallback.enabled        |

#### circuitBreaker断路器

`circuitBreaker.enabled`: 该属性用来确定当服务请求命令失败时， 是否使用断路器来跟踪其健康指标和熔断请求。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.command.default.circuitBreaker.enabled               |
| 实例默认值   | @HystrixProperty(name=”circutBreaker.enabled”,value=”false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.circuitBreaker.enabled     |

`circuitBreaker.requestVolumeThreshold`: 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为20 的时候，如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 20                                                           |
| 全局配置     | hystrix.command.default.circuitBreaker.requestVolumeThreshold |
| 实例默认值   | @HystrixProperty(name=”circuitBreaker.requestVolumeThreshold”, value=”30”) |
| 实例配置     | hystrix.comrnand.HystrixComrnandKey.circuitBreaker.requestVolumeThreshold |

`circuitBreaker.sleepWindowinMilliseconds`: 该属性用来设置当断路器打开之后的休眠时间窗。休眠时间窗结束之后，会将断路器置为“半开” 状态， 尝试熔断的请求命令，如果依然失败就将断路器继续设置为“打开” 状态，如果成功就设置为“关闭” 状态。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 5000                                                         |
| 全局配置     | hystrix.command.default.circuitBreaker.sleepWindowinMilliseconds |
| 实例默认值   | @HystrixProperty(name=”circuitBreaker.sleepWindowinMilliseconds”,value=”3000”) |
| 实例配置     | hystrix.command.HystrixCommandKey.circuitBreaker.sleepWindowinMilliseconds |

`circuitBreaker.errorThresholdPercentage`: 该属性用来设置断路器打开的错误百分比条件。例如，默认值为5000 的情况下，表示在滚动时间窗中，在请求数量超过`circuitBreaker.requestVolumeThreshold`阅值的前提下，如果错误请求数的百分比超过50, 就把断路器设置为“打开” 状态， 否则就设置为“关闭” 状态。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 50                                                           |
| 全局配置     | hystrix.command.default.circuitBreaker.errorThresholdPercentage |
| 实例默认值   | @HystrixProperty(name=”circuitBreaker.errorThresholdPercentage”, value=”40”) |
| 实例配置     | hystrix.command.HystrixCommandKey.circuitBreaker.errorThresholdPercentage |

`circuitBreaker.forceOpen`: 如果将该属性设置为true, 断路器将强制进入“打开” 状态，它会拒绝所有请求。该属性优先于`circuitBreaker.forceClosed`属性。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | false                                                        |
| 全局配置     | hystrix.command.default.circuitBreaker.forceOpen             |
| 实例默认值   | @HystrixProperty (name=”circuitBreaker.forceOpen”, value=”true”) |
| 实例配置     | hystrix.command.HystrixCommandKey.circuitBreaker.forceOpen   |

`circuitBreaker.forceClosed`: 如果将该属性设置为true, 断路器将强制进入“关闭” 状态， 它会接收所有请求。如果`circuitBreaker.forceOpen`属性为true, 该属性不会生效。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | false                                                        |
| 全局配置     | hystrix.command.default.circuitBreaker.forceClosed           |
| 实例默认值   | @HystrixProperty(name=”circui七Breaker.forceClosed”, value=”true”) |
| 实例配置     | hystrix.comrnand.HystrixComrnandKey.circuitBreaker.forceClosed |

#### metrics配置

该配置与HystrixCommand执行中捕获的指标信息有关。

`metrics.rollingStats.timeinMilliseconds`: 该属性用来设置滚动时间窗的长度， 单位为毫秒。该时间用于断路器判断健康度时需要收集信息的持续时间。断路器在收集指标信息的时候会根据设置的时间窗长度拆分成多个“桶” 来累计各度量值，每个“桶” 记录了一段时间内的采集指标。例如，当采用默认值10000毫秒时， 断路器默认将其拆分成10个桶（桶的数量也可通过`metrics.rollingStats.numBuckets`参数设置），每个桶记录1000毫秒内的指标信息。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 10000                                                        |
| 全局配置     | hystrix.command.default.metrics.rollingStats.timeinMilliseconds |
| 实例默认值   | @HystrixProperty(name=”metrics.rollingStats.timeinMilliseconds”,value=”20000”) |
| 实例配置     | hystrix.command.HystrixCommandKey.metrics.rollingStats.timeinMilliseconds |

`metrics.rollingstats.numBuckets`: 该属性用来设置滚动时间窗统计指标信息时划分“桶” 的数量。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 10                                                           |
| 全局配置     | hystrix.command.default.metrics.rollingStats.numBuckets      |
| 实例默认值   | @HystrixProperty(name=”metrics.rollingStats.numBuckets”,value=”20”) |
| 实例配置     | hystrix.comrnand.HystrixComrnandKey.metrics.rollingStats.numBuckets |

`metrics.rollingPercentile.enabled`: 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为false，那么所有的概要统计都将返回-1。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.command.default.metrics.rollingPercentile.enabled    |
| 实例默认值   | @HystrixProperty(name=”metrics.rollingPercentile.enabled”, value=”false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.metrics.rollingPercentile.enabled |

`metrics.rollingPercentile.timeinMilliseconds`: 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 60000                                                        |
| 全局配置     | hystrix.command.default.metrics.rollingPercentile.timeinMilliseconds |
| 实例默认值   | @HystrixProperty(name=”metrics.rollingPercentile.timeinMilliseconds”, value=”50000”) |
| 实例配置     | hystrix.command.HystrixCommandKey.metrics.rollingPercentile.timeinMilliseconds |

`metrics.rollingPercentile.numBuckets`: 该属性用来设置百分位统计滚动窗口中使用“桶”的数量。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 6                                                            |
| 全局配置     | hystrix.command.default.metrics.rollingPercentile.numBuckets |
| 实例默认值   | @HystrixProperty(name=”metrics.rollingPercentilee.numBuckets”,value=”5”) |
| 实例配置     | hystrix.command.HystrixCommandKey.metrics.rollingPercentile.numBuckets |

`metrics.rollingPercentile.bucketSize`: 该属性用来设置在执行过程中每个“桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个“桶”中发生了500次执行，那么该“桶”中只保留最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 100                                                          |
| 全局配置     | hystrix.command.default.metrics.rollingPercentile.bucketSize |
| 实例默认值   | @HystrixProperty(name=”metrics.rollingPercentile.bucketSize”,value= “120”) |
| 实例配置     | hystrix.command.HystrixCommandKey.metrics.rollingPercentile.bucketSize |

`metrics.healthSnapshot.intervalinMilliseconds`: 该属性用来设置采集影响断路器状态的健康快照（请求的成功、错误百分比）的间隔等待时间。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 500                                                          |
| 全局配置     | hystrix.comrnand.default.metrics.healthSnapshot.intervalinMilliseconds |
| 实例默认值   | @HystrixProperty(name=”metrics.healthSnapshot.intervalinMilliseconds”,value=”600”) |
| 实例配置     | hystrix.command.HystrixCommandKey.metrics.healthSnapshot.intervalinMilliseconds |

#### requestContext

`requestCache.enabled`: 此属性用来配置是否开启请求缓存。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.command.default.requestCache.enabled                 |
| 实例默认值   | @HystrixProperty(name=”requestCache.enabled”, value=”false”) |
| 实例配置     | hystrix.command.HystrixCommandKey.requestCache.enabled       |

### collapser

`maxRequestsinBatch`: 该参数用来设置一次请求合并批处理中允许的最大请求数。

| 属性级别     | 默认值、配置方式、配置属性                                |
| :----------- | :-------------------------------------------------------- |
| 全局默认配置 | Integer.MAX_VALUE                                         |
| 全局配置     | hystrix.collapser.default.maxRequestsinBatch              |
| 实例默认值   | @HystrixProperty(name=”maxRequestsinBatch”,value=”false”) |
| 实例配置     | hystrix.collapser.HystrixCollapserKey.maxRequestsinBatch  |

`timerDelayinMilliseconds`: 该参数用来设置批处理过程中每个命令延迟的时间，单位为毫秒。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 10                                                           |
| 全局配置     | hystrix.collapser.default.timerDelayinMilliseconds           |
| 实例默认值   | @HystrixProperty(name=”timerDelayinMilliseconds”,value=”20”) |
| 实例配置     | hystrix.collapser.HystrixCollapserKey.timerDelayinMilliseconds |

`request Cache.enabled`: 该参数用来设置批处理过程中是否开启请求缓存。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | true                                                         |
| 全局配置     | hystrix.collapser.default.requestCache.enabled               |
| 实例默认值   | @HystrixProperty(name=”requestCache.enabled”, value=”false”) |
| 实例配置     | hystrix.collapser.HystrixCollapserKey.requestCache.enabled   |

### threadPool

`coreSize`: 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量。

| 属性级别     | 默认值、配置方式、配置属性                       |
| :----------- | :----------------------------------------------- |
| 全局默认配置 | 10                                               |
| 全局配置     | hystrix.threadpool.default.coreSize              |
| 实例默认值   | @HystrixProperty(name=”coreSize”, value=”false”) |
| 实例配置     | hystrix.threadpool.HystrixThreadPoolKey.coreSize |

`maxQueueSize`: 该参数用来设置线程池的最大队列大小。当设置为-1时，线程池将使用`SynchronousQueue`实现的队列，否则将使用`LinkedBlockingQueue`实现的队列。

| 属性级别     | 默认值、配置方式、配置属性                           |
| :----------- | :--------------------------------------------------- |
| 全局默认配置 | -1                                                   |
| 全局配置     | hystrix.threadpool.default.maxQueueSize              |
| 实例默认值   | @HystrixProperty(name=”maxQueueSize”,value=”lO”)     |
| 实例配置     | hystrix.threadpool.HystrixThreadPoolKey.maxQueueSize |

`queueSizeRejectionThreshold`: 该参数用来为队列设置拒绝阈值。通过该参数，即使队列没有达到最大值也能拒绝请求。该参数主要是对`LinkedBlockingQueue`队列的补充， 因为`LinkedBlockingQueue`队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。

| 属性级别     | 默认值、配置方式、配置属性                                   |
| :----------- | :----------------------------------------------------------- |
| 全局默认配置 | 5                                                            |
| 全局配置     | hystrix.threadpool.default.queueSizeRejectionThreshold       |
| 实例默认值   | @HystrixProperty(name=”queueSizeRejectionThreshold”, value=”lO” |
| 实例配置     | hystrix.threadpool.HystrixThreadPoolKey.queueSizeRejectionThreshold |



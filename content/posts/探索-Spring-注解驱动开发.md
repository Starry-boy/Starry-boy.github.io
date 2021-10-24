---
title: "Spring 注解驱动开发"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
 - Spring
 - 注解
keywords:
 - Spring
 - 注解
description: 
---

# 探索 Spring 注解驱动开发
## 一、注解

- @Configuration

  用来替代 application.xml 配置文件，注解上自带 @Component 代表被该注解修饰的类，也会直接被 Spring 管理

- @ComponentScan

  注解扫描 扫描包下所有类 将和符合条件的类由 Spring 容器管理，可以自定义配置 bean 注册规则 

- @Bean

  表示，该方法的返回对象会注册到 Spring 容器中，方法名 就是bean在容器中的名字

- @Scope	

  设置组件作用范围，默认为单例配置为多例时，Bean 的初始化变成懒加载，Request：每次请求都会创建一个实例，Session：同一个 Session 创建一个实例

- @Lazy	

  懒加载，第一次获取 Bean 时，才会加载 Bean，只针对 单例，多例 默认懒加载

- @Conditional	

  自定义条件，符合条件才会注册 Bean

- @Import

  1. 直接填 Class 对象：将参数中的 class对象全部实例化 注册进容器

  2. ImportSelector：这是个接口，实现 selectImports 返回需要注册的对象的类全名数组

     ```java
     public interface ImportSelector {
     		//AnnotationMetadata 中能获取被 @Import 中带有 ImportSelector 修饰的类上的所有注解
         String[] selectImports(AnnotationMetadata var1);
     
         @Nullable
         default Predicate<String> getExclusionFilter() {
             return null;
         }
     }
     ```

  3. ImportBeanDefinitionRegistrar

     > AnnotationMetadata：能获取被 @Import 中带有 ImportBeanDefinitionRegistrar 修饰的类上的所有注解
     >
     > BeanDefinitionRegistry：可以通过 BeanDefinitionRegistry 收入注册 Bean

     ```java
     public interface ImportBeanDefinitionRegistrar {
         default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {
             this.registerBeanDefinitions(importingClassMetadata, registry);
         }
     
         default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
         }
     }
     ```

  

## 二、注册Bean的方式

- @ComponentScan + @Component/@Controller/@Service/@Repository/@Configuration

- @Bean

- @Import

- BeanDefinitionRegistry

- 实现 FactoryBean

  注册工厂

  ```java
      @Bean
      public RatelFactoryBean ratelFactoryBean(){
          return new RatelFactoryBean();
      }
  ```

  工厂实现

  ```java
  public class RatelFactoryBean implements FactoryBean<User> {
      public User getObject() throws Exception {
          return new User();
      }
  
      public Class<?> getObjectType() {
          return User.class;
      }
  }
  ```

  通过 "ratelFactoryBean" 的名字向 IOC 容器获取出来的 RatelFactoryBean 通过调用 getObject 获取的 User 对象，因为 工厂 Bean 在注册时，会在名字前加上前缀 &，所有要获取 RatelFactoryBean 需要提供的 Bean 的名字应该为 &RatelFactoryBean

  

## 三、Bean 的生命周期

1. 创建

   - 构造方法

2. 初始化

   - `@Bean(initMethod = "")`

   - 实现 InitializingBean 接口

     > Bean 初始化属性注入完成后，会调用 afterPropertiesSet 方法

   - JSR250 `@PostConstruct`

     > Bean 初始化属性注入完成后，执行

   - 实现 BeanPostProcessor 接口的 postProcessBeforeInitialization 方法

     > Bean 初始化之前执行

   - 实现 BeanPostProcessor 接口的 postProcessAfterInitialization 方法

     > Bean 初始化之后执行

3. 销毁

   - `@Bean(destroyMethod = "")`

     > 调用 applicationContext.destroy() 方法时容器会被销毁，被容器管理的 Bean 会调用 `@Bean(destroyMethod = "")` 中的方法，如果 Bean 是多例的 则不会调用销毁方法

   - 实现 DisposableBean 接口

     > 容器销毁时 会调用 Bean 的 destroy 方法

   - JSR250 @PreDestroy

     > 容器销毁 前 执行

   

## 四、获取 SpringContext

- 实现 ApplicationContextAware 接口

  > 重写 setApplicationContext 方法，在方法入参中 即可获得 ApplicationContext 对象

  ```java
  public interface ApplicationContextAware extends Aware {
      void setApplicationContext(ApplicationContext var1) throws BeansException;
  }
  ```

## 五、赋值、注入

- @Value

  1. 基本数值

  2. SpEL：#{}

  3. 取配置文件值（properties）（运行时环境变量中的值）：${}

     > 可以直接获取 ConfigurableEnvironment environment = applicationContext.getEnvironment(); 中的数据

- @PropertySource

  读取配置文件中的数据 （properties 文件）

- @PropertySources

  读取配置文件中的数据 （properties 文件）

- @Autowired

  从容器获取按类型获取实例 注入到属性中，或者方法入参中，如果只有一个有参构造方法，则可以省略 @Autowired，Bean 在初始化时 还是会从容器中获取 参数注入

- @Qualifier

  配合 @Autowired 使用，容器中有多个同类型的 Bean 时,可以通过 @Qualifier 指定 按名字注入

- @Primary

  容器中有多个同类型的 Bean 时,优先注入 @Primary 修饰的 Bean

- @Resource  (JSR250 规范)

  作用和 @Autowired 一样，默认按 名称注入，且不支持 @Primary 指定默认 Bean

- @Inject (JSR303)

  作用和 @Autowired 一样，需要导入 javax.inject 包，不支持 required=false

- @Profile

  根据当前环境，动态的激活或切换一系列组件的功能，默认激活 @Profile("default")

  可以修饰在方法和类上，修饰在方法上：不满足激活条件时 不会执行该方法，修饰在类上：不满足条件时 不会执行 该类中的所有 @Bean 修饰的方法

  <font color='red'>切换环境</font>

  1. 命令行参数激活： -Dspring.profile.active=<?>

  2. 代码激活

     ```java
     //1. 获取 applicationContext
     //省略
     //2. 指定需要激活的环境
     applicationContext.getEnvironment().setActiveProfiles("dev","test");
     //3.设置主配置类 (配置类 使用自己写的配置类)
     applicationContext.register(SpringAnnotationConfiguration.class);
     //4.启动刷新容器
     applicationContext.refresh();
     ```

- 实现 **Aware 接口


# Spring源码解读

## 重要的对象

- BeanDefinition

  Spring 中对 Bean 的描述，是一中数据结构，在 Spring 中 BeanDefinition 和 Bean 的关系，类似 Java 中 Object 和 Class 的关系。Class 是对 Object 的描述，BeanDefinition 是对 Spring Bean 的描述

- BeanFactory

  俗称 Spring容器，实例化 Bean 并管理 Bean

- internalConfigurationAnnotationProcessor

- ConfigurationClassPostProcessor

- AutowiredAnnotationBeanPostProcessor

- CommonAnnotationBeanPostProcessor

- EventListenerMethodProcessor

- DefaultEventListenerFactory

## 重要方法

- AbstractApplicationContext#refresh
  1. 通过 register 注入普通 Bean 时不需要刷新容器
  2. 通过 register 注入 Configuration Bean 时 需要刷新容器
- 

视频学习 1:15:50
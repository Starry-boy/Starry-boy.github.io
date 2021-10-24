# Dubbo源码解析：服务暴露

## 服务提供者配置

- 配置截图

  通过XML的形式配置 dubbo 提供者，主要使用到的是 Spring 的自定义标签功能

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122119125963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

- XML 配置解析

  - DubboNamespaceHandler

    初始化 Dubbo 标签的解析器。解析 dubbo:service、dubbo:reference 等标签

    每个解析器对应一个标签,服务提供者对应 service 标签，所以重点关注 ServiceBean 这个类

    
  
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221192319513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221192637760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

    

    parse 方法

    将 XML 标签转换为 BeanDefinition 对象,然后待 Spring 根据 BeanDefinition 对象去创建对应的代理对象

    

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116155551128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

    

  - ServiceBean

    

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221194509996.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

    

    实现了Spring常见的3个接口

    - InitialzingBean
  
      **Bean初始化成功会调用 afterPropertiesSet 方法**
  
      主要作用 获取 dubbo 基础配置
  
      ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122119521355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)
  
    - ApplicationContextAware
  
      Bean 初始化成功后会调用 setApplicationContext 方法，通过此方法可以获取到 SpringContext 对象
  
      获取 SpringContext 后添加了一个监听器
  
      ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221195440648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)
    
    - ApplicationListener
    
      在容器启动成功Bean都初始化完成后调用 onApplicationEvent 
    
    


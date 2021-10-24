---
title: "ELK 分布式日式分析系统"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Log
tags: 
  - LogStash
  - EasticSearch
  - Kibana
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - LogStash
  - EasticSearch
  - Kibana
description: 
---

# ELK 分布式日式分析系统

## 安装

安装 ElasticSearch

```
docker pull elasticsearch
```

安装 Kibana

```
docker pull kibana:7.7.0
```

安装 LogStash

```
docker pull logstash:7.7.0
```

## 启动

<font color='red'>注意:</font> 

1. Es 默认堆内存为2G,如果内存过小 需要限制内存
2. Es 启动需要加上discovery 等环境变量，不过不加启动会报错，可以通过docker 日志查看错误信息
3. Kibana 默认访问Es 的URL= http://elasticsearch:9200 需要修改为 ES的 URL，否则访问会 Kibana server is not ready yet

### Elasticsearch

```
docker run -d -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" --name ElasticSearch elasticsearch:7.7.0
```

### Kibana

```
docker run -d -p 5601:5601 -e ELASTICSEARCH_URL="http://172.18.0.8:9200" --name Kibana kibana
```

### LogStash

```
docker run -d -p 5044:5044 --memory 500m  --name Logstash 30dcca1db5e9
```

## 解决 Elasticsearch 跨域问题

> 修改 yaml 配置文件

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 修改 LogStash 配置

<font color='red'>注意：docker 方式部署需要 修改logstash.yml 指定读取的配置文件</font>

```yaml
http.host: "0.0.0.0"
path.config: /usr/share/logstash/config/logstash-sample.conf
# xpack.monitoring.elasticsearch.hosts: [ "http://172.17.0.3:9200" ]
```



```yaml
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
  tcp {
    port => 5044
    mode => "server"
    #codec => json_lines
  }
}

output { 
	stdout { 
		codec => rubydebug 
	}
  elasticsearch {
    hosts => ["http://172.18.0.8:9200"]
    index => "es-message-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
}
```

##  修改 Logback 配置文件

```yaml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="./logs" />
    <!-- 控制台输出 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
    <!-- 按照每天生成日志文件 -->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5044</destination>
        <includeCallerData>true</includeCallerData>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder" />
        <!--<encoder-->
                <!--class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">-->
            <!--<providers>-->
                <!--<timestamp>-->
                    <!--<timeZone>Asia/Shanghai</timeZone>-->
                <!--</timestamp>-->
                <!--<pattern>-->
                    <!--<pattern>-->
                        <!--{-->
                        <!--"level": "%-5level",-->
                        <!--"message": "%msg",-->
                        <!--"time": "%d{yyyy-MM-dd HH:mm:ss.SSS}",-->
                        <!--"logger":"%logger",-->
                        <!--"appName":"${log_name}"-->
                        <!--}-->
                    <!--</pattern>-->
                <!--</pattern>-->
            <!--</providers>-->
        <!--</encoder>-->
    </appender>
    <!-- 日志输出级别 -->
    <root level="DEBUG">
        <appender-ref ref="console" />
        <appender-ref ref="file" />
        <appender-ref ref="logstash" />
    </root>
</configuration>
```

## 修改 KIbana 配置文件

```yaml
#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://172.18.0.8:9200" ]
monitoring.ui.container.elasticsearch.enabled: true

```

## 测试

1. 启动 ElasticSearch、Logstash、和 Web应用

2. 访问 ElsaticSearch  http://localhost:9200

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520171257908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

3. 访问 Kibana http://localhoist:5601

   <font color='red'>注意：首次访问 Kibana 是没有数据的，需要手动指定索引</font>

   [Kibana 使用教程](https://jingyan.baidu.com/article/aa6a2c14a269ce0d4c19c4a5.html)

   

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520171444235.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)
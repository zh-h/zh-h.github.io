---
title: RefreshScope 生效条件
date: 2017-12-10
tags: [Spring Cloud]
categories: [Java]
---

## Spring Cloud Config 更新配置
Spring Cloud Netflix Bus是Spring Cloud的消息机制,当Git Repository 改变时,通过POST请求Config Server的/bus/refresh,Config Server 会从repository获取最新的信息并通过amqp传递给client,如图所示.

![https://segmentfault.com/img/bVAhXA?w=1429&h=580](https://segmentfault.com/img/bVAhXA?w=1429&h=580)

Spring Cloud Bus的更新只对三种情况有效

1. @ConfigurationProperties
2. @RefreshScope
3. 日志级别

## @ConfigurationProperties

使用 Java 定义类匹配配置文件中的声明
应用代码
```groovy
@ConfigurationProperties(prefix='api')
class Api{
    String host
    Integer port
}

class ApiService{
    @Autowrired
    Api api

    @Autowired
    RestTemplate restTemplate

    def hello(){
        restTemplate.getForObject("${api.host}:${api.port}", String)
    }
}
```
配置文件
```yml
api:
    host: 127.0.0.1
    port: 6666
```
当配置中心更新了配置文件后，会通过消息通知客户端拉取新的配置，@ConfigurationProperties 注解的类会及时得到更新。

## @RefreshScope
如果涉及消息队列或者数据库连接的配置，就需要声明 @RefreshScope

声明 @RefreshScope 后如果使用了注入的 property 类或者使用 @Value('${a.b,c}') 语法取配置，收到刷新配置的消息时，将重新初始化类。

```
@Component
@RefreshScope
@RabbitHandler(queues='${rcs.mq.strategyResponse}')
class StrategyLisener{
    void hander(String message){
        // TODO
    }
}
```

刷新配置将会重新初始化这个类，并且监听的是新配置的队列名。

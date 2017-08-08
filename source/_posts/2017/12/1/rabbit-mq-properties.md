---
title: RabbitMQ 的消息属性说明
date: 2017-12-01
tags: [RabbitMQ]
categories: [Java]
---

## RabbitMQ
AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。
AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。


## RabbiMQ  RPC 应用
MQ本身是基于异步的消息处理，前面的示例中所有的生产者（P）将消息发送到RabbitMQ后不会知道消费者（C）处理成功或者失败（甚至连有没有消费者来处理这条消息都不知道）。

但实际的应用场景中，我们很可能需要一些同步处理，需要同步等待服务端将我的消息处理完成后再进行下一步处理。这相当于RPC（Remote Procedure Call，远程过程调用）。在RabbitMQ中也支持RPC。
![http://cdndiggerplus.b0.upaiyun.com/wp-files/2014/02/2014-2-21-9-59-04.png](http://cdndiggerplus.b0.upaiyun.com/wp-files/2014/02/2014-2-21-9-59-04.png)


- 客户端发送请求（消息）时，在消息的属性（MessageProperties，在AMQP协议中定义了14中properties，这些属性会随着消息一起发送）中设置两个值replyTo（一个Queue名称，用于告诉服务器处理完成后将通知我的消息发送到这个Queue中）和correlationId（此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个id了解哪条请求被成功执行了或执行失败）
- 服务器端收到消息并处理
- 服务器端处理完消息后，将生成一条应答消息到replyTo指定的Queue，同时带上correlationId属性
- 客户端之前已订阅replyTo指定的Queue，从中收到服务器的应答消息后，根据其中的correlationId属性分析哪条请求被执行了，根据执行结果进行后续业务处理


## RabbitMQ AMQP properties 规范
在开发环境中，需要使用 RabbitMQ 的管理面板进行消息的模拟发送，契约规定了微服务相互间的序列化使用 JSON，但是陷入了苦恼之中。

在表单中填写 properties 为 ContentType:application/json 无效！Spring AMQP 依然会把消息当作默认的类型进行处理，也就是对象流！点击源码一步步跳转深入发现，解析 properties 键值对的过程中，键并不是类似于 HTTP 的 Header 使用连字符，而是使用下划线，并且是小写！

就是 content_type:application/json 才会当作 json 类型的消息进行解析。


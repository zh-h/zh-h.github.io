---
title: 负载均衡原理
date: 2017-08-21
tags: [架构]
categories: [Java]
---

## 说明

负载均衡对物理、虚拟服务器或者独立应用进行网络流量的分发，可以包括传输层和应用程的处理。

负载均衡可以通过流量分发扩展应用系统对外的服务能力，通过消除单点故障提升应用系统的可用性。

## 负载均衡策略

1. Round Robin
    对所有可用后端进行循环访问。
2. Least Connections(least_conn)
    跟踪记录后端的连接数，优先选择链接数少的服务，将请求转发，并且设计每个upstream分配的weight权重信息。
3. Least Time(least_time)
    请求会分配给响应最快和活跃连接数最少的后端。
4. IP Hash(ip_hash)
    对请求来源IP地址计算hash值，然后根据得到的hash值通过某种映射分配到后端（Nginx常用）。
5. Generic Hash(hash)
    以用户自定义资源(比如URL)的方式计算hash值完成分配，其可选consistent关键字支持一致性hash特性；

## 常见负载均衡实现

### Nginx

使用 Nginx 进行负载均有以下特点：

1. 运行在应用层，对HTTP应用进行分流。

2. 常用IP hash进行回话绑定。

3. 反向代理可以实现缓存。

缺点：
1. 支持的应用协议少，只支持HTTP、HTTPS和WebSocket等。

2. 支持应用端口的可用性检查。

### Haproxy

优点：

1. 运行于四层网络，可以进行TCP协议的负载均衡，如数据库的°负载均衡。

2. 比其他负载均衡软件高的性能。

### DNS

通常应用在多地多级房，多运营商网络的负载均衡。

## 健康检查

### keepalived

// TODO
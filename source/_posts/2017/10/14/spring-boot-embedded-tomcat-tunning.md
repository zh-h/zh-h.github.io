---
title: Spring Boot 内嵌 tomcat 调优
date: 2017-10-14
tags: [Spring,Tomcat]
categories: [Java]
---

## 前言
为了能够承受更多的用户量，并且改善性能，需要对 Sevlet 容器的最大连接数、最大线程数、IO模式进行设置。

## 外部 Tomcat 参数调优

略。

## Spring Boot 内嵌 tomcat 参数调优

面试被问到达这个问题，一般都是使用默认参数，还没有实际调整的经验，于是回答了 JVM 的命令行参数。。。

### Spring Boot 配置属性

以下是默认的 Tomcat 容器在 Spring Boot 中的配置属性

`application.properties`

```
server.tomcat.accesslog.directory=logs # Directory in which log files are created. Can be relative to
the tomcat base dir or absolute.
server.tomcat.accesslog.enabled=false # Enable access log.
server.tomcat.accesslog.pattern=common # Format pattern for access logs.
server.tomcat.accesslog.prefix=access_log # Log file name prefix.
server.tomcat.accesslog.rename-on-rotate=false # Defer inclusion of the date stamp in the file name
until rotate time.
server.tomcat.accesslog.request-attributes-enabled=false # Set request attributes for IP address,
Hostname, protocol and port used for the request.
server.tomcat.accesslog.suffix=.log # Log file name suffix.
server.tomcat.background-processor-delay=30 # Delay in seconds between the invocation of
backgroundProcess methods.
server.tomcat.basedir= # Tomcat base directory. If not specified a temporary directory will be used.
server.tomcat.internal-proxies=10\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}|\\
192\\.168\\.\\d{1,3}\\.\\d{1,3}|\\
169\\.254\\.\\d{1,3}\\.\\d{1,3}|\\
127\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}|\\
172\\.1[6-9]{1}\\.\\d{1,3}\\.\\d{1,3}|\\
172\\.2[0-9]{1}\\.\\d{1,3}\\.\\d{1,3}|\\
172\\.3[0-1]{1}\\.\\d{1,3}\\.\\d{1,3} # regular expression matching trusted IP addresses.
server.tomcat.max-threads=0 # Maximum amount of worker threads. 最大线程数
server.tomcat.min-spare-threads=0 # Minimum amount of worker threads. 最小线程数
server.tomcat.port-header=X-Forwarded-Port # Name of the HTTP header used to override the original port
value.
server.tomcat.protocol-header= # Header that holds the incoming protocol, usually named "X-ForwardedProto".
server.tomcat.protocol-header-https-value=https # Value of the protocol header that indicates that the
incoming request uses SSL.
server.tomcat.redirect-context-root= # Whether requests to the context root should be redirected by
appending a / to the path.
server.tomcat.remote-ip-header= # Name of the http header from which the remote ip is extracted. For
instance `X-FORWARDED-FOR`
server.tomcat.uri-encoding=UTF-8 # Character encoding to use to decode the URI.
```

我们能从中进行调整的主要是`max-threads`和`min-spare-threads`两个参数。

### TomcatEmbeddedServletContainerFactory
但是我们需要设置 Tomcat 的最大连接数呢？不像独立的 Tomcat 可以配置`server.xml`文件，Spring Boot 内置的 Tomcat 支持使用 Java 编码进行配置，可以重写`TomcatEmbeddedServletContainerFactory`

```java
package me.zonghua.tomcat.tunning;

import org.apache.catalina.connector.Connector;
import org.apache.coyote.http11.Http11NioProtocol;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.stereotype.Component;


@Component
public class CustomEmbeddedServletContainerFactory extends TomcatEmbeddedServletContainerFactory {
    private static final Logger LOGGER = LoggerFactory.getLogger(CustomEmbeddedServletContainerFactory.class.getName());
    @Value("${server.tomcat.max-connections}")
    private Integer maxConnections = 1024;

    @Override
    protected void customizeConnector(Connector connector) {
        super.customizeConnector(connector);
        super.customizeConnector(connector);
        Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
        protocol.setMaxConnections(maxConnections);  //设置最大连接数,请他参数请参考 protocol 的属性
        LOGGER.info("Embedded tomcat max connections set: " + maxConnections);
    }
}
```

以下是可以支持更改的属性：

0. maxConnections: 这个值表示最多可以有多少个socket连接到tomcat上。NIO模式下默认是10000.

1. maxThreads： Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数。默认值150。

2. acceptCount： 指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。默认值10。

3. minSpareThreads： Tomcat初始化时创建的线程数。默认值25。

4. maxSpareThreads： 一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。默认值75。

5. enableLookups： 是否反查域名，默认值为true。为了提高处理能力，应设置为false

6. connnectionTimeout： 网络连接超时，默认值60000，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。

7. maxKeepAliveRequests： 保持请求数量，默认值100。 bufferSize： 输入流缓冲大小，默认值2048 bytes。

8. compression： 压缩传输，取值on/off/force，默认值off。 其中和最大连接数相关的参数为maxThreads和acceptCount。如果要加大并发连接数，应同时加大这两个参数。


### JVM 参数

可以使用 Maven 插件指定打包后 jar 运行的默认 JVM 参数。

`pom.xml`

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
                <jvmArguments>-Xms512m -Xmx512m -XX:NewSize=64m -XX:MaxNewSize=512m -XX:PermSize=256m</jvmArguments>
            </configuration>
        </plugin>
    </plugins>
</build>
```

JVM 调优是一门玄学，如果掌握后那就是高级工程师了。。。

[https://www.cubrid.org/blog/how-to-tune-java-garbage-collection](https://www.cubrid.org/blog/how-to-tune-java-garbage-collection)

![tomcat tunning](https://www.cubrid.org/files/attach/images/1730/731/001/117f4cc137a4b953815933dcd51f481b.png)




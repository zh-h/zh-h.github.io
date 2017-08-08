---
title: Spring Boot 注册为 Linux 系统服务
date: 2017-09-25
tags: [Spring,Linux]
categories: [Linux]
---

## 服务管理
Spring Boot 不需要依赖外部 Servlet 容器，可以直接通过`java -jar app.jar`的方式执行。

在一些情况下不适用 Docker 容器管理 Spring Boot 进程的时候，需要使用其他的进程管理工具管理，如`systemctl`需要编写管理脚本。

实际上 Maven 插件打包 Spring Boot 的时候可以直接生成管理脚本，然后直接使用服务管理。

## 插件配置

Maven 配置
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

Gradle 配置

```groovy
apply plugin: 'org.springframework.boot'

springBoot {
    executable = true
}
```

## 建立软链接

经过打包的可执行脚本被内置在Spring Boot jar包里，链接到`/etc/init.d`。

可以将应用安装在/var/myapp, 使用下面命令将Spring Boot应用作为init.d服务。

```bash
chmod a+x /var/myapp/myapp.jar
ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```

## 管理进程

注册到到服务后管理进程将十分方便。

```bash
/etc/init.d/myapp [start|stop|restart]
```

进程的PID在`/var/run/myapp/myapp.pid`

日志被重定向到`/var/log/myapp.log`
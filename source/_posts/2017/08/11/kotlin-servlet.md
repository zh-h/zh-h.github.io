---
title: Kotlin Servlet 示例
date: 2017-08-11
tags: [Kotlin,Servlet]
categories: [Kotlin]
---

## 添加 Servlet 依赖

添加 JaveEE 的API，包含了 Servlet 的引用。

```groovy
dependencies {
    compile group: 'javax', name: 'javaee-api', version: '7.0'
    compile "org.jetbrains.kotlin:kotlin-stdlib-jre8:$kotlin_version"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

## Gradle Tomcat Plugin

**使用 jetty plugin 无法加载注解的 servlet，请使用 tomcat plugin**

1. 编辑`gradle.build`,附加 tomcat 插件，配置启动任务。

```groovy
apply plugin: "war"
apply plugin: 'com.bmuschko.tomcat'

buildscript {
    dependencies {
        classpath 'com.bmuschko:gradle-tomcat-plugin:2.3'
    }
}

dependencies {
    def tomcatVersion = '8.5.16'
    tomcat "org.apache.tomcat.embed:tomcat-embed-core:${tomcatVersion}",
            "org.apache.tomcat.embed:tomcat-embed-logging-juli:8.5.2",
            "org.apache.tomcat.embed:tomcat-embed-jasper:${tomcatVersion}"
}

tomcat {
    httpProtocol = 'org.apache.coyote.http11.Http11Nio2Protocol'
    ajpProtocol = 'org.apache.coyote.ajp.AjpNio2Protocol'
    contextPath = ''
}
```

2. 创建`src/main/webapp`目录，添加`index.html`文件，编辑内容。

```html
<html>
<head>
    <title>Hello Kotlin</title>
</head>
<body>
    <h1>Hello Kotlin</h1>
</body>
</html>
```

## 添加控制器

添加`me.zohar.kotlin.servlet.HelloController`，这个控制器继承了`HttpServlet`,并且调用默认的父类的构造方法。

```kotlin
package me.zohar.kotlin.servlet.controller

import javax.servlet.annotation.WebServlet
import javax.servlet.http.HttpServlet
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse

@WebServlet(name = "Hello", value = "/")
class HomeController : HttpServlet() {
    override fun doGet(req: HttpServletRequest, res: HttpServletResponse) {
        res.writer.write("Hello, World!")
    }
}
```

## 访问

`http://localhost:8080/`

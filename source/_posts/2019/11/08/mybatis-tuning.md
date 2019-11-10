---
title: mybatis tuning
date: 2019-11-08
tags: [mybatis,sql]
categories: [java]
---

// TODO 获取连接时间
// TODO 真正执行时间

## 打印生成完整SQL输出
在 `application.yml` 添加日志配置
```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```
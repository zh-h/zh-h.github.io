---
title: iPhone Fiddler HTTPS 抓包
date: 2017-09-19
tags: [计算机网络]
categories: [Linux]
---

## 前期准备

1. iOS设备和电脑保证在一个网段内，相互间可联通。

2. 下载安装 Fiddler

## 配置

启动 Fiddler 以下截图为 Windows 平台，其他平台 Fiddler 可做参考。

### 解码 HTTPS

打开 Tool > Options > HTTPS 

设置 Fiddler 解码 HTTPS 内容

![http://wx2.sinaimg.cn/large/e7c91439gy1fjp3iuk3whj20f50a9gm1.jpg](http://wx2.sinaimg.cn/large/e7c91439gy1fjp3iuk3whj20f50a9gm1.jpg)


### 开启远程访问

打开 Tool > Options > Connections

勾选 Allow remote computers to connect


### 重启 Fiddler
重启 Fiddler 代理服务器才会生效


![http://wx1.sinaimg.cn/large/e7c91439gy1fjp3iuj1dzj20f40a8q39.jpg](http://wx1.sinaimg.cn/large/e7c91439gy1fjp3iuj1dzj20f40a8q39.jpg)

### 添加代理设置

查看电脑的内网IP

打开 iOS 设备的无线局域网 > 代理设置

填写IP和默认的端口 8888

![http://wx3.sinaimg.cn/large/e7c91439gy1fjp3ivnp9gj20jz0zk40t.jpg](http://wx3.sinaimg.cn/large/e7c91439gy1fjp3ivnp9gj20jz0zk40t.jpg)

打开 http://ip:8888

点击 Fiddler certificate 

![http://wx4.sinaimg.cn/large/e7c91439gy1fjp3iws18hj20jz0zkabp.jpg](http://wx4.sinaimg.cn/large/e7c91439gy1fjp3iws18hj20jz0zkabp.jpg)

安装认证证书

![http://wx4.sinaimg.cn/large/e7c91439gy1fjp3iuil1ej20jz0zkgmg.jpg](http://wx4.sinaimg.cn/large/e7c91439gy1fjp3iuil1ej20jz0zkgmg.jpg)

打开 iOS 设备的通用 > 关于本机

使证书可信

![http://wx3.sinaimg.cn/large/e7c91439gy1fjp3iuigvrj20jz0zkgmg.jpg](http://wx3.sinaimg.cn/large/e7c91439gy1fjp3iuigvrj20jz0zkgmg.jpg)

## 示例-访问微信公众号文章阅读数

1. 打开微信应用，点击访问任意文章

![http://wx3.sinaimg.cn/large/e7c91439gy1fjp447j65bj20jz0zk0uq.jpg](http://wx3.sinaimg.cn/large/e7c91439gy1fjp447j65bj20jz0zk0uq.jpg)

2. 检查 Fiddler 左侧回话列表

3. 点击带 getappmsgext 的 url

4. 右侧面板上面是 HTTP 头部内容

5. 右侧面板下部是 HTTP 正文

6. 点击 TextView 选项卡

7. 点击上面提示的 Response body is encoded, click to decode 提示

8. 查看解码后的正文内容

![http://wx4.sinaimg.cn/large/e7c91439gy1fjp3iv3pkxj20im0bt752.jpg](http://wx4.sinaimg.cn/large/e7c91439gy1fjp3iv3pkxj20im0bt752.jpg)

9. 流程

```
+-------+            +------------------+            +------+
|       |            |                  |            |      |
|       |  request   |   intercept      |  request   |      |
|       |  +----->   |                  |   +---->   |      |
|       |            |   dencrypt       |            |      |
|       |  response  |                  |  response  |      |
|       |  <----+    |   decode         |   <-----+  |      |
|       |            |                  |            |      |
|       |            |   parse          |            |      |
|       |            |                  |            |      |
+-------+            +------------------+            +------+
 iPhone                   Fidder                      Website
```




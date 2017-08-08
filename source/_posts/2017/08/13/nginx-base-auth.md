---
title: Nginx 启用 Base Auth 实现用户认证
date: 2017-08-13
tags: [Nginx]
categories: [Linux]
---


如果一些 Web 系统（如：Kibana）没有提供用户认证管功能，可以使用 Nginx 反向代理启用 Base Auth。

**Base Auth安全性低，不检验在外网直接使用，请务必添加SSL**

## 安装 httpd-tools

```
apt-get install apache2-utils
```

## 生成用户密码

```
htpasswd -c /etc/nginx/.htpasswd admin  
```

## 修改 Nginx 配置

针对 Nginx 配置不同的上下文，可以针对服务器或者目录进行验证：

`nginx.conf`

```
location / {  
                auth_basic "HTTP Basic Authentication";  
                auth_basic_user_file /etc/nginx/.htpasswd;  
        }  
```

## 重载 Nginx

```
service nginx reload
```

## 在 Dockerfile 中配置

本示例基础镜像以 Ubuntu 或者 Debian 为基准，编辑 `Dockerfile`

```
ENV BASE_AUTH=admin:adminadmin

RUN apt-get install apache2-utils -y && \
    mkdir -p /etc/nginx && \
    echo $BASE_AUTH > /etc/nginx/.htpasswd
```

`docker-nginx.conf`

```
        auth_basic "HTTP Basic Authentication";  
        auth_basic_user_file /etc/nginx/.htpasswd; 

        location ~ \.php$ {
            try_files $uri /index.php =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass php-fpm:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
            include /etc/nginx/fastcgi_params;
        }
```

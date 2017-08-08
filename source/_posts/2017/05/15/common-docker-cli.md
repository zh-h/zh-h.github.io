---
title: 开发环境常用的 Docker 镜像
date: 2017-05-15
categories: [Docker,Linux]
---

## Portainer

```
docker run -d -p 10000:9000 --name=portainer --restart=always -v "/var/run/docker.sock:/var/run/docker.sock"  portainer/portainer
```

## Postgresql

```
docker run --name docker-postgres -e POSTGRES_PASSWORD=adminadmin -e POSTGRES_USER=root -p 5432:5432 --restart=always -d postgres:9.6-alpine
```

## Pgadmin4

```
docker run -d  --name docker-pgadmin4 --restart always -p 5050:5050 --link docker-postgres  chorss/docker-pgadmin4
```

## MySQL

```
docker run --name docker-mysql --restart always -e MYSQL_ROOT_PASSWORD=adminadmin  -p 3306:3306 -d mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```
## PhpMyAdmin

```
docker run --name docker-phpmyadmin -d --link docker-mysql:db -p 3307:80 phpmyadmin/phpmyadmin
```

## Redis

```
docker run --name docker-redis  -d -p 6379:6379 --restart always  redis:3.2.9-alpine
```

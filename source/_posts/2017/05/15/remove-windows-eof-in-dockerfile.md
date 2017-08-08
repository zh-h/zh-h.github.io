---
title: Docker 构建时移除文件中 Windows 换行符
date: 2017-05-15 15:05:00
categories: [Docker,Linux]
---

## 换行符

Windows 上的换行符是 `\r\n`;

*nix 系统中使用 `\n`。

如果在 Windows 上修改，甚至通过 Git 等版本控制工具移动文本文件，都会导致换行符的改变。

如果换行符变更，很多脚本将会运行报错，如果在 Linux 系统中使用 `vi` 编辑器查看，会每行后面看到 `^M` 字符。

## Docker 运行出错

在 Windows 中使用 Docker 作为开发环境，如果启动容器的时候发现脚本执行出现错误，就要注意是否换行符导致的错误。如：
```
: not found| docker-entrypoint.sh: 2: docker-entrypoint.sh:
dockertypecho_php-fpm_1 exited with code 127
```

```

```

## 替换 Windows 换行符

### `sed` 命令

在构建镜像的时候，使用 `sed` 命令把将要执行的脚本文件中的换行符全部替换。

`Dockerfile`

```
ADD ./src/docker-entrypoint.sh /var/www/html/docker-entrypoint.sh
RUN sed -i 's/\r//g' docker-entrypoint.sh
```
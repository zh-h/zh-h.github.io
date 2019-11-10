---
title: docker 容器内存、内存管理和事故
date: 2019-11-11
tags: [linux,docker]
categories: [docker]
---

## 获取容器内真正内存使用

### 验证
编写一个程序不断申请并释放内存
```python
import logging
import os

logging.basicConfig(format='%(levelname)s %(asctime)s %(filename)s %(funcName)s %(lineno)s: %(message)s', level=logging.INFO)

SIZE = 128 * 1024 * 1024


def print_memory():
    with os.popen('/bin/ps -eo pid,rss') as result:
        lines = result.readlines()
        total_memory = 0
        for line in lines:
            columns = line.split(' ')
            porcess_memory = columns[len(columns) - 1]
            porcess_memory = porcess_memory.strip()
            try:
                total_memory = total_memory + int(porcess_memory)
                logging.debug('total momory %d', total_memory)
            except ValueError as e:
                logging.debug(e)
                pass
        logging.info('%.2fMiB', total_memory/1024)
        return total_memory
    return 0

def allocate_memory(size):
    bytes = []
    for v,i in enumerate(range(size)):
        bytes.append(1)
    logging.info('%.2fMiB', len(bytes)/1024/1024)
    print_memory()

if __name__ == "__main__":
    while True:
        allocate_memory(SIZE)
```
执行输出
```bash
INFO 2019-11-10 18:41:28,615 x.py allocate_memory 31: 128.00MiB
INFO 2019-11-10 18:41:28,972 x.py print_memory 23: 1038.19MiB
INFO 2019-11-10 18:41:37,819 x.py allocate_memory 31: 128.00MiB
INFO 2019-11-10 18:41:37,836 x.py print_memory 23: 1038.41MiB
INFO 2019-11-10 18:41:46,665 x.py allocate_memory 31: 128.00MiB
```
可以看到内存不断申请，内存上涨，然后到了一定大小后就会触发内存回收，内存减少

### 错误的方法

### 1. `/proc/meminfo` 命令
使用`/proc/meminfo`获取到的硬件信息，是宿主机的内存

newrelic的代理端就是使用这样的命令获取系统信息，在容器内就会出现隐患。

JVM默认的最大Heap大小是系统内存的1/4，假若物理机内存为10G，如果你不手动指定Heap大小，则JVM默认Heap大小就为2.5G。

JavaSE8(<8u131)版本前还没有针对在容器内执行高度受限的Linux进程进行优化，JDK1.9 以后开始正式支持容器环境中的CGroups内存限制，JDK1.10 这个功能已经默认开启，可以查看相关Issue（Issue地址：https://bugs.openjdk.java.net/browse/JDK-8146115）。

JVM Heap是一个只增不减的内存模型，Heap的内存只会往上涨，不会下降。在容器里面使用Java，如果为JVM未设置Heap大小，Heap取得的是宿主机的内存大小，当Heap的大小达到容器内存大小时候，就会触发系统对容器OOM，Java进程会异常退出。

使用`/proc/[pid]/statm` 来获取进程使用的内存情况。

### 2. `free` 命令
`free` 命令的数据就是从`/proc/meminfo`计算得到的，实际上还是宿主机内存。

### 正确方法
使用`ps`命令获取容器内所有进程以及内存信息
把所有运行的进程的内存汇总加起来
#### 1. python
调用`ps`命令
```python
import logging
import os

logging.basicConfig(format='%(levelname)s %(asctime)s %(filename)s %(funcName)s %(lineno)s: %(message)s', level=logging.INFO)

def print_memory():
    with os.popen('/bin/ps -eo pid,rss') as result:
        lines = result.readlines()
        total_memory = 0
        for line in lines:
            columns = line.split(' ')
            porcess_memory = columns[len(columns) - 1]
            porcess_memory = porcess_memory.strip()
            try:
                total_memory = total_memory + int(porcess_memory)
                logging.debug('total momory %d', total_memory)
            except ValueError as e:
                logging.debug(e)
                pass
        logging.info('%.2fMiB', total_memory/1024)
        return total_memory
    return 0
```
### 2. java
调用`ps`命令
```java
    private Long getTotalMemory() throws IOException {
        Process process = Runtime.getRuntime().exec("/bin/ps -eo pid,rss");
        InputStreamReader inputStreamReader = new InputStreamReader(process.getInputStream());
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        StringBuilder stringBuilder = new StringBuilder();
        while (true) {
            String line = bufferedReader.readLine();
            if (line == null) {
                break;
            }
            stringBuilder.append(line);
            stringBuilder.append("\n");
        }
        logger.debug(stringBuilder.toString());
        String outPut = stringBuilder.toString();
        String[] lines = outPut.split("\n");
        Long totalMemory = 0L;
        for (String line : lines) {
            String[] columns = line.split(" ");
            String processMemoryString = columns[columns.length - 1];
            try {
                Long processMemory = Long.valueOf(processMemoryString);
                totalMemory += processMemory;
            } catch (Exception e) {
                // pass
            }
        }
        logger.info("{}", totalMemory);
        return totalMemory;
    }
```

### 3. docker
`docker stats`命令是通过对cgroup中相关数据进行取值从而计算得到内存使用和CPU使用等参数

参考：https://www.cnblogs.com/xuxinkun/p/5541894.html
```bash
docker stats 76482df813d4
CONTAINER ID        NAME                   CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
76482df813d4        affectionate_pasteur   100.40%             71.45MiB / 1.952GiB   3.57%               1.11kB / 0B         1.61MB / 0B         
```
---
title: NodeMCU with MicroPython
date: 2017-10-19
tags: [python,IoT]
categories: [Linux]
---

## 刷入固件

### 准备

0. 使用 USB 连接 NodeMCU 到 PC
1. 下载 MicroPython for ESP8266 的固件
2. 下载安装 CP210x_Windows_Drivers
3. 使用 pip 安装 esptool 和 ampy

### 擦除Flash

1. 在设备管理器中找到串口连接串口号（比如 COM3）
2. 运行`esptool.py --port COM3 erase_flash`把先前固件的数据都删掉，防止出现一些问题
运行正确的话输出如下
```bash
$ esptool.py --port COM3 erase_flash
esptool.py v2.1
Connecting....
Detecting chip type... ESP8266
Chip is ESP8266
Uploading stub...
Running stub...
Stub running...
Erasing flash (this may take a while)...
Chip erase completed successfully in 10.6s
Hard resetting...
```

### 刷入

1. 运行`esptool.py --port COM3 write_flash -fm qio 0x00000 esp8266-20170823-v1.9.2.bin`
默认的`-fm`写入模式是`dio`对于大于 4M 的 Flash 会有更好的兼容，但是速率受限。
> multi I/O SPI设备是有从单一设备支持增加带宽或throughput的能力。相对于标准的串行Flash存储设备，一个dual I/O接口能够使能双倍的速率。quad I/O接口能提升throughput四次。

2. 运行成功的输出如下
```bash
$ esptool.py --port COM3 write_flash -fm qio 0x00000 esp8266-20170823-v1.9.2.bin
8266-20170823-v1.9.2.bin
esptool.py v2.1
Connecting....
Detecting chip type... ESP8266
Chip is ESP8266
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
Auto-detected Flash size: 4MB
Flash params set to 0x0240
Compressed 601136 bytes to 392067...
Wrote 601136 bytes (392067 compressed) at 0x00000000 in 34.7 seconds (effective 138.7 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting...
```
3. 按下板子的 RST 键重启

## REPL
> REPL — 交互式解释器环境。
R(read)、E(evaluate)、P(print)、L(loop) 
输入值，交互式解释器会读取输入内容并对它求值，再返回结果，并重复此过程。

Micropython 提供REPL，但是为了能够人机交互，需要其他的软件接收信号发送给开发板。

### putty

1. 设置串口号和波特率

![设置串口号和波特率](http://wx4.sinaimg.cn/large/e7c91439ly1fknxp6kjmqj20ck0c5myu.jpg)

2. 设置其他参数

![设置其他参数](http://wx2.sinaimg.cn/large/e7c91439ly1fknxp6h5s0j20ck0c5t9y.jpg)

3. 打开连接
```bash
l▒"rdr$r▒n▒▒▒▒▒oܟ▒▒#r▒▒b쏜#$▒#$▒▒l#$▒▒ln▒p{l▒l▒▒|▒▒▒#4 ets_task(40100164, 3, 3fff837c, 4)
OSError: [Errno 2] ENOENT

MicroPython v1.9.2-8-gbf8f45cf on 2017-08-23; ESP module with ESP8266
Type "help()" for more information.
>>> help()
Welcome to MicroPython!
```

### ampy

Adafruit MicroPython Tool 通过串口连接 MicroPython 单板的的工具，支持获取、罗列、删除、提交文件以及运行脚本的功能。

1. 运行`ampy --port COM3 get boot.py`获取`boot.py`的内容
```
$ ampy --port COM3 get boot.py
# This file is executed on every boot (including wake-boot from deepsleep)
#import esp
#esp.osdebug(None)
import gc
#import webrepl
#webrepl.start()
gc.collect()
```
或者设置终端的环境变量`export AMPY_PORT=COM3`(Windows CMD: set AMPY_PORT=COM3)指定端口
```
$ ampy get boot.py
```

### node-serialport
一个 Node.js 库，支持Linux、Windows和MacOS，使用 Javascript 事件驱动编程，支持命令行打开模拟终端连接串口

1. 罗列所有串口
```bash
PS C:\>serialport-term.cmd
COM3    USB\VID_10C4&PID_EA60\0001      Silicon Labs
```
2. 进入终端
```bash
PS C:\>serialport-term.cmd -p COM3 -b 115200
l▒"rdr$r▒n▒▒▒▒▒oܟ▒▒#r▒▒b쏜#$▒#$▒▒l#$▒▒ln▒p{l▒l▒▒|▒▒▒#4 ets_task(40100164, 3, 3fff837c, 4)
OSError: [Errno 2] ENOENT

MicroPython v1.9.2-8-gbf8f45cf on 2017-08-23; ESP module with ESP8266
Type "help()" for more information.
```

## 使用 MicroPython 进行编程
// TODO

## 串口编程

### 使用 node-serialport
1. 使用`process.stdin.setRawMod(true)`获取所有按键输入，包括控制按键等,实现串口连接的模拟[终端](#REPL)
```javascript
const PORT_NAME = 'COM3'
const BAUD_RATE = 115200 // config those

const colors = require('colors')
const SerialPort = require('serialport');
const port = new SerialPort(PORT_NAME, {
    baudRate: 115200,
    autoOpen: false,
    rtscts: true
});

const itOut = (data) => {
    process.stdout.write(data.toString())
}

const itError = (data) => {
    process.stdout.write(data.toString())
}
port.open(e => {
    port.write('import os;os.listdir()\r') // '\r' is return means next line, in Windows is '\r\n'

    port.on('error', err => {
        itError(err)
    })

    port.on('end', (s) => {
        port.flush()
        port.close();
    })


    port.on('readable', function () {
        itOut(port.read())
    });

    process.stdin.setRawMode(true)
    let exitCounter = 0
    process.stdin.on('data', (s) => {
        if (s[0] === 0x03) {
            if (exitCounter == 1) {
                itOut('bye\n')
                port.close();
                process.exit(0);
            } else {
                itOut('Wanna exit? ctrl+c continue.\n')
                port.write(s)  // ctrl+c 终止当前运行的程序
                exitCounter++;
            }
        } else {
            exitCounter = 0
            port.write(s)
        }
        if (s[0] === 0x0d) {
            itOut('\n')
        }
    });
})

```

## non-blocking input | raw input
**类似 node.js process.stdin.setRawMode(true) 的操作**，不需要回车，获取所有包括ctrl+B等不可见字符
- Python
```python
# 参考 Django createsuperuser 的时候输入密码的功能
from sys import stdout
import platform

if platform.system() == 'Windows':
    is_windows = True
    import msvcrt


    def getch_loop():
        while True:
            key = msvcrt.getch()
            key_ascii = int.from_bytes(key, byteorder='little')
            stdout.write(chr(key_ascii))  # unicode
            stdout.write('\n')
            stdout.flush()
            if key_ascii == 3:
                print('bye')
                break
else:
    is_windows = False
    import curses

    window = curses.initscr()
    curses.noecho()
    curses.cbreak()
    window.keypad(True)


    def getch_loop():
        try:
            while True:
                key_ascii = window.getch()
                stdout.write(chr(key_ascii))  # unicode
                stdout.write('\r\n')
                stdout.flush()
        except KeyboardInterrupt:
            print('bye')
            curses.endwin()
        except Exception:
            curses.endwin()

if __name__ == '__main__':
    getch_loop()

```
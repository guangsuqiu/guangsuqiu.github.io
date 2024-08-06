---
title: 编译交叉编译工具gdb，安装gdb-multiarch，交叉编译gdbserver，配置C语言VSCode调试环境 教程
date: 2024-08-06 11:11:07
tags: ["C语言","gdb","VS Code","交叉编译"]
description:
categories: "Linux"
---
​
# 前言

在嵌入式开发中，调试程序是优秀开发人员的必备技能，今天分享一下gdb源码从编译到vscode调试全过程。

# 一、获取/编译交叉编译工具gdb
## 方式1-源码编译
### 1.  从GNU站点下载GDB源码
[源码地址](http://ftp.gnu.org/gnu/gdb/)中提供了各个版本的gdb源码，可以选择点击下载，在linux下，可以选择使用wget进行下载。本文选择在linux命令行使用wget下载8.2版本，具体命令如下：
```bash
wget http://ftp.gnu.org/gnu/gdb/gdb-8.2.tar.gz
```
### 2. 解压文件
```bash
tar -xzf gdb-8.2.tar.gz
```
我们可以在工程顶层目录下发现configure脚本文件，在此需要介绍一下该文件的一些经常使用的入参的含义，其他参数意义可输入`./configure --help`来查看。
| 参数 | 含义| 
|:----|:----|
|--prefix|指定安装路径，在make install阶段将生成的程序放入指定目录|
|--host|指定软件运行的系统平台，如果没有指定将会运行`config.guess`来检测|
|--target|生成的程序可以在本地运行，程序编译会生成指定平台的程序，场景：编译交叉编译工具gcc，gdb|
|--build|正常执行编译的平台，如果不指定将会运行`config.guess`来检测|

这里做一个示例方便大家理解，假设我们有一台x86_64的Linux虚拟机，一个Arm的开发板。
- 场景一：在虚拟机中编译虚拟机使用的gdb
	```bash
	./configure 
	```
	其中`--host` `--target` `--build`让 `config.guess`来检测|，扩展后如下所示
	```bash
	./configure --build=x86_64-pc-linux-gnu --host=x86_64-pc-linux-gnu --target=x86_64-pc-linux-gnu
	```
- 场景二：在虚拟机中编译虚拟机使用的，调试支持arm平台的gdb
	```bash
	./configure --target=arm-sanechips-linux-gnueabi
	```
	其中`--host` `--build`让 `config.guess`来检测|，扩展后如下所示
	```bash
	./configure --build=x86_64-pc-linux-gnu --host=x86_64-pc-linux-gnu --target=arm-sanechips-linux-gnueabi
	```
- 场景三：在虚拟机中编译arm平台下运行的gdb
	```bash
	./configure --host=arm-sanechips-linux-gnueabi CC=arm-sanechips-linux-gnueabi-gcc
	```
	其中`--target` `--build`让 `config.guess`来检测|，扩展后如下所示
	```bash
	./configure --build=x86_64-pc-linux-gnu --host=arm-sanechips-linux-gnueabi --target=arm-sanechips-linux-gnueabi CC=arm-sanechips-linux-gnueabi-gcc
	```

### 3. 生成交叉编译工具gdb
由于默认编译的GDB在调试时会出现`Remote ‘g’ packet reply is too long`的错误，我们需要修改GDB的源码
在`./gdb/remote.c`文件中搜索以下内容
```bash
  if (buf_len > 2 * rsa->sizeof_g_packet)
    error (_("Remote 'g' packet reply is too long (expected %ld bytes, got %d "
	     "bytes): %s"), rsa->sizeof_g_packet, buf_len / 2, rs->buf);
```
将上文内容替换为
```c
	if (buf_len > 2 * rsa->sizeof_g_packet)
	{
	    rsa->sizeof_g_packet = buf_len;
	    for (i = 0; i < gdbarch_num_regs (gdbarch); i++){
	    if (rsa->regs[i].pnum == -1)
	    	continue;
	    if (rsa->regs[i].offset >= rsa->sizeof_g_packet)
	    	rsa->regs[i].in_g_packet = 0;
	    else
	    	rsa->regs[i].in_g_packet = 1;
	    }
	}
```
在gdb工程顶层目录下输入以下命令，运行后生成Makefile
```bash
./configure --target=arm-sanechips-linux-gnueabi --prefix=/home/guangsuqiu/Desktop/gdb-8.2/build
```
之后输入以下命令，则在`/home/guangsuqiu/Desktop/gdb-8.2/build/bin`目录下生成`arm-sanechips-linux-gnueabi-gdb`
```bash
make -j8 && make install
```
至此交叉编译工具gdb已编译完成，可输入`./arm-sanechips-linux-gnueabi-gdb -v`查看信息。
```bash
guangsuqiu@Ubuntu1804:~/Desktop/gdb-8.2/build/bin$ ./arm-sanechips-linux-gnueabi-gdb -v
GNU gdb (GDB) 8.2
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```


## 方式2-安装gdb-multiarch
`gdb-multiarch`是一款一款支持多个 Arch 的 gdb 调试工具，安装后不需要做特殊的配置即可使用。
在Ubuntu1804版本下，可使用apt命令进行安装。
```bash
sudo apt install gdb-multiarch
```
# 二、交叉编译gdbserver

## 1. 设置交叉编译工具链环境变量
`cd`到交叉编译工具链中的`bin`文件夹，确保里面存放了`XXX-XXX-gcc`类似的文件，我的文件名为`arm-sanechips-linux-gnueabi-gcc`。输入以下命令设置临时环境变量，仅该终端有效。
```bash
export PATH=$PWD:$PATH
```
## 2.交叉编译gdbserver
在同一个终端下，`cd`到gdb工程目录，执行以下命令
```bash
cd ./gdb/gdbserver/
./configure --host=arm-sanechips-linux-gnueabi CC=arm-sanechips-linux-gnueabi-gcc --prefix=/home/guangsuqiu/Desktop/gdb-8.2/build
make -j8 && make install
```
在`/home/guangsuqiu/Desktop/gdb-8.2/build/bin`下生成gdbserver
可以使用file命令查看信息
```bash
guangsuqiu@Ubuntu1804:~/Desktop/gdb-8.2/build/bin$ file gdbserver 
gdbserver: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.3, for GNU/Linux 5.4.0, BuildID[sha1]=36d554d9fc5bd1459029a69d9d9ab65c70e1a9b9, with debug_info, not stripped
```

# 三、配置VSCode环境
## 1. 配置调试环境
在VSCode侧边栏`运行和调试`中`添加调试配置`，在文件中填充以下内容
```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gdb-XXX",
            "type": "cppdbg",
            "request": "launch",
            // ${workspaceFolder}值vscode顶层目录
            "program": "${workspaceFolder}/test",
            // 参数在设备测gdbserver设置。e.g. gdbserver 192.168.10.3:6666 test -D
            "args": [""],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            // 使用gdbserver时启用
            // 如果是自编译的gdb，路径需要变化
            "miDebuggerPath":"/usr/bin/gdb-multiarch",
            // gdbserver 的ip地址及端口号
            "miDebuggerServerAddress": "192.168.10.1:6666"
        }
    ]
}
```

## 2. 配置自动任务
### (1). 创建tasks.json文件
vscode界面中，选择菜单中`终端`选项，选择`配置任务`，选择`使用模板创建tasks.json文件`，选择`MSBuild 执行生成目标`。

### (2). 修改tasks.json文件
```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build test",
            "type": "shell",
            // 该命令会在vscode顶层目录运行
            "command": "make && make install",
            "group": "build",
            "presentation": {
                // Reveal the output only if unrecognized errors occur.
                "reveal": "silent"
            },
            // Use the standard MS compiler pattern to detect errors, warnings and infos
            "problemMatcher": "$msCompile"
        }
    ]
}
```

### (3). 运行任务
VSCode界面中，选择菜单中`终端`选项，选择`运行任务`，选择`build test`

# 四、补充
## 1. 关于gdb
如果编译出的gdb连接gdbserver后出现`warning: Can not parse XML target description; XML support was disabled at compile time`警告，通常是因为编译时没有找到XML的解析库expat，这不影响正常使用，如果介意警告的朋友，可以参考[gdbserver 调试时gdb运行c时崩溃不能正常调试](https://blog.csdn.net/yangzhongxuan/article/details/13002789)博客进行重新编译。
## 2. 关于调试信息，strip程序与编译时-s的作用
在gcc编译时，添加`-s`选项会删除调试信息，即使有`-g`存在也不能进行调试。
程序编译完成后，也可以使用`对应编译工具的strip`进行裁剪，将调试信息去掉。
这里展示一下裁剪与未裁剪的区别
```bash
# 未裁剪调试信息，最后显示not stripped
$ arm-sanechips-linux-gnueabi-gcc -g  main.c
$ file a.out 
a.out: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.3, for GNU/Linux 5.4.0, BuildID[sha1]=52f66c47eec336c93d90ddf020d668a058a57a27, with debug_info, not stripped
# 裁剪调试信息后，最后显示stripped
# 编译时使用-s
$ arm-sanechips-linux-gnueabi-gcc -g -s main.c
# 编译后使用strip裁剪
$ arm-sanechips-linux-gnueabi-gcc -g  main.c
$ arm-sanechips-linux-gnueabi-strip a.out
$ file a.out 
a.out: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.3, for GNU/Linux 5.4.0, BuildID[sha1]=ec5dd2942630715e9f0ea1608a7c9cc6d18757fd, stripped
```
## 3. gdb与gdbserver调试
gdb与gdbserver调试时，参数由gdbserver控制
```bash
./gdbserver 192.168.10.3:6666 test AB
```
# 五、参考链接
1. [交叉编译GDB工具链](https://segmentfault.com/a/1190000021029824)
2. [configure 配置参数说明](https://blog.csdn.net/xhoufei2010/article/details/82768995)
3. [利用VS Code+Qemu+GDB调试Linux内核](http://rainlin.top/archives/201)
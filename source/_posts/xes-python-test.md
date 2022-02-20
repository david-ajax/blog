---
title: 在学而思的临时 Linux 环境上测试程序
date: 2022-2-20
---
对于一般的单文件 Python 程序, 可以使用学而思编程通过在线 IDE 提供的的临时 Linux 服务器上进行测试.
<!--more-->
# 服务运行技术
经测试, 学而思编程提供的 Python IDE 使用两种技术运行服务  
## 使用 Docker 技术在 GNU/Linux 服务器内创建多个 Debian 系 Linux 容器并在容器内执行代码.
经测试, 内核版本是 Linux 4.19, 架构是 x86_64. 无法使用 root 权限与 sudo 可以运行大部分 Linux 二进制包  
默认用户名为 *dev*, 无法使用 bash 作为 shell(只是被禁用了, 可以绕过), 可以使用 cd, ls, whoami, chmod, chown等命令, 但可以通过 dir 与 find 替换 ls, 无法使用 apt 安装软件, 但自带有 gcc clang python2 python3, 均可运行, 且可以使用 pip 安装包.  
容器可以联网, 但无法执行 wget curl lynx 等命令, 可以自己用 python 造轮子
### 在容器内使用 Bash 的方法
```bash
$ cd /bin
$ find -name "*bash*"
bash
rbash
8iibiauewf83guewf87238......bash(类似于这种形式)
$ 8iibiauewf83guewf87238......bash
dev@hostname:~/bin$ cd
dev@hostname:~$ 
```

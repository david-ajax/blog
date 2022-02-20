---
title: 在学而思的临时 Linux 环境上测试程序并练习 Linux Shell 操作
date: 2022-2-19
---
当没有 Linux 环境时, 可以使用学而思编程通过在线 IDE 提供的的临时 Linux 服务器上进行临时测试和演示代码.  
在某些情况下格外有用.  
由此篇文章所引起的一切问题与作者无关.  
<!--more-->
# 服务运行技术
经测试, 学而思编程提供的 Python IDE 使用两种技术运行服务:  
1. 使用 Docker 技术在 GNU/Linux 服务器内创建 Debian 系 Linux 容器并在容器内执行代码.(Python 基础模式与 C++)
2. 在本地安装"学而思编程助手", 实则是一个 Python 环境, 由网页调用 API, 在本地执行程序.(其他 Python 模式)
我们要使用在线的环境, 所以选择 "Python 基础模式".
当然, 也可以使用 C++ 模式, 通过 system() 函数调用 shell.
经测试, 学而思服务器内核版本是 Linux 4.19, 架构是 x86_64. 可能是为了防止滥用,无法使用 root 权限与 sudo 但可以运行大部分无需 root 权限的 Linux 二进制包.  
默认用户名为 *dev*, 可以使用 bash 作为 shell(但默认被禁用, 可以通过 sh 绕过).  
可以使用 cd, ls, whoami, chmod, chown等命令, 但可以通过 dir 与 find 替换 ls, 无法使用 apt 安装软件, 但自带有 gcc clang python2 python3, 均可运行, 且可以使用 pip 安装包.  
可以安装 busybox.  
容器可以联网, 但无法执行 wget curl lynx 等命令, 可以自己用 python 造轮子.  
# 在容器内使用 Bash 的方法
在左侧编辑框内输入:
```python
import os
os.system("sh")
```
运行后, 右侧终端会启动一个交互式 shell(sh), 在里面操作:
```bash
$ cd /bin
$ find -name "*bash*"
bash
rbash
8iibiauewf83guewf87238......bash(类似于这种形式)
$ 8iibiauewf83guewf87238......bash
dev@hostname:~/bin$ cd
dev@hostname:~$ echo "成功进入 Bash"
```
现在, 我们已经成功进入 Bash.
但这还远远不够, 我们需要一个更方便的方式来调用系统操作.  
# 安装 Busybox
此处用 $ 指代 Linux Shell
```bash
$ pip3 install wget
$ python3
>>> import wget, os
>>> os.system("mkdir busybox")
>>> os.chdir("busybox")
>>> wget.download("https://www.busybox.net/downloads/binaries/1.35.0-x86_64-linux-musl/busybox", "busybox") # 下载 busybox
>>> os.system("chmod 777 busybox")
>>> os.system("export PATH=$(pwd):$PATH") # 添加环境变量
>>> os.system("..")
>>> exit()
$ busybox
BusyBox v1.35.0 multi-call binary.
BusyBox is copyrighted by many authors between 1998-2015.
Licensed under GPLv2. See source distribution for detailed
copyright notices.

Usage: busybox [function [arguments]...]
   or: busybox --list[-full]
   or: busybox --install [-s] [DIR]
   or: function [arguments]...

        BusyBox is a multi-call binary that combines many common Unix
        utilities into a single executable.  The shell in this build
        is configured to run built-in utilities without $PATH search.
        You do not need to install a link to busybox for each utility.
        To run external program, use full path (/sbin/ip instead of ip).

Currently defined functions:
        [, [[, acpid, adjtimex, ar, arch, arp, arping, ash, awk, basename, bc, blkdiscard, blockdev, brctl, bunzip2, busybox, bzcat, bzip2, cal, cat, chgrp, chmod, chown, chpasswd,
        chroot, chvt, clear, cmp, cp, cpio, crond, crontab, cttyhack, cut, date, dc, dd, deallocvt, depmod, devmem, df, diff, dirname, dmesg, dnsdomainname, dos2unix, dpkg,
        dpkg-deb, du, dumpkmap, dumpleases, echo, ed, egrep, env, expand, expr, factor, fallocate, false, fatattr, fdisk, fgrep, find, fold, free, freeramdisk, fsfreeze, fstrim,
        ftpget, ftpput, getopt, getty, grep, groups, gunzip, gzip, halt, head, hexdump, hostid, hostname, httpd, hwclock, i2cdetect, i2cdump, i2cget, i2cset, id, ifconfig, ifdown,
        ifup, init, insmod, ionice, ip, ipcalc, ipneigh, kill, killall, klogd, last, less, link, linux32, linux64, linuxrc, ln, loadfont, loadkmap, logger, login, logname, logread,
        losetup, ls, lsmod, lsscsi, lzcat, lzma, lzop, md5sum, mdev, microcom, mkdir, mkdosfs, mke2fs, mkfifo, mknod, mkpasswd, mkswap, mktemp, modinfo, modprobe, more, mount, mt,
        mv, nameif, nc, netstat, nl, nologin, nproc, nsenter, nslookup, nuke, od, openvt, partprobe, passwd, paste, patch, pidof, ping, ping6, pivot_root, poweroff, printf, ps, pwd,
        rdate, readlink, realpath, reboot, renice, reset, resume, rev, rm, rmdir, rmmod, route, rpm, rpm2cpio, run-init, run-parts, sed, seq, setkeycodes, setpriv, setsid, sh,
        sha1sum, sha256sum, sha512sum, shred, shuf, sleep, sort, ssl_client, start-stop-daemon, stat, static-sh, strings, stty, su, sulogin, svc, svok, swapoff, swapon, switch_root,
        sync, sysctl, syslogd, tac, tail, tar, taskset, tc, tee, telnet, telnetd, test, tftp, time, timeout, top, touch, tr, traceroute, traceroute6, true, truncate, tty, tunctl,
        ubirename, udhcpc, udhcpd, uevent, umount, uname, uncompress, unexpand, uniq, unix2dos, unlink, unlzma, unshare, unxz, unzip, uptime, usleep, uudecode, uuencode, vconfig,
        vi, w, watch, watchdog, wc, wget, which, who, whoami, xargs, xxd, xz, xzcat, yes, zcat
$ busybox wget
BusyBox v1.35.0 multi-call binary.

Usage: wget [-c|--continue] [--spider] [-q|--quiet] [-O|--output-document FILE]
        [--header 'header: value'] [-Y|--proxy on/off] [-P DIR]
        [--no-check-certificate]
        [-S|--server-response] [-U|--user-agent AGENT] URL...
```
现在, 我们可以执行操作了, 注意调用命令前要加上 busybox.  
你可以把自己的代码放于 Github 上, 然后用 busybox 下载并解压 zip 包, 即可用 gcc, clang 与 python 调试并打包程序.  
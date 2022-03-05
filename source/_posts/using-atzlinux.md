---
title: 使用铜豌豆 Linux 软件源以在 Ubuntu Linux 上安装迅雷, QQ, 微信等众多常用软件
date: 2022-3-6
---
> 铜豌豆 Linux 收集整理制作了大量中国人常用的应用软件, 丰富 Linux 桌面系统软件生态......
<!--more-->
# 添加铜豌豆 Linux 软件源
在 shell 上运行以下命令:  
```bash
sudo apt -y install wget coreutils
sudo wget -c -O atzlinux-archive-keyring_lastest_all.deb https://www.atzlinux.com/atzlinux/pool/main/a/atzlinux-archive-keyring/atzlinux-archive-keyring_lastest_all.deb
sudo apt -y install ./atzlinux-archive-keyring_lastest_all.deb
sudo dpkg --add-architecture i386
sudo apt update
```
然后运行  
```bash
sudo apt install 包名 -y -f
```
以安装你的软件  
安装迅雷:  
```bash
sudo apt install com.xunlei.download -y -f
```
安装微信(wine):  
```bash
sudo apt install com.qq.weixin.deepin -y -f
```
安装 QQ:  
```bash
sudo apt install deepin.com.qq.im
```
安装 QQ 音乐:  
```bash
sudo apt install qqmusic
```
安装 TIM:  
```bash
sudo apt install deepin.com.qq.office
```
查看完整列表, 请参考 https://www.atzlinux.com/allpackages.htm  
# 部分问题
## 找不到 sudo?
这种问题一般发生在极为干净的 rootfs 里  
切入 root 用户 (因为没有sudo)  
安装 coreutils  
```bash
apt install coreutils -y -f
```
然后通过 visudo 命令修改 /etc/sudoers (注意要在 root 环境下编辑, 且不用附带参数)  
visudo 会自动检测你的配置文件格式, 以避免 sudo 无法使用  
```bash
visudo
```
然后这样修改:  
```bash
.............省略.............
root    ALL=(ALL)       ALL
ajax   ALL=(ALL)       ALL #把上面 root 的行复制粘贴，把 root 改成你想使用的用户名(如 ajax)
```
## 找不到 apt?
考虑一下安装 ubuntu 或 debian  
此博客仅支持 debian 系发行版  
当然, 你也可以在不改变你当前系统的情况下, 使用 chroot, docker 等容器技术运行 debian/ubuntu 容器, 然后用 vnc 或其他方式连接 GUI  
由于此操作过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过过于繁琐, 所以不再赘述  
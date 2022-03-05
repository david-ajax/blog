---
title: 获取网易云音乐的直接下载地址
date: 2022-3-16
---
如题
<!--more-->
# 获得解析链接
首先在客户端或网站上获取歌曲的播放地址, 通常是像这样: http://music.163.com/#/m/song?id=5221167  
根据链接的基础知识, 我们可以把这个 URI 拆分成几个部分:  
Prorocol: http:  
Auth: none  
Hostname(Domain): music.163.com  
Port: none(实际为80)  
/#/m/song - Pathname  
id=5221167 - Quary(不带问号)  
我们要用到 Quary 部分  
将其嵌入解析链接  
格式: https://music.163.com/song/media/outer/url?[quary].mp3  
比如 http://music.163.com/#/m/song?id=5221167  
就是 https://music.163.com/song/media/outer/url?id=5221167.mp3  
# 嵌入到 HTML
先来听听效果:  
Never Gonna Give You Up - Rick Astley  
<audio controls="controls" title="Never Gonna Give You Up" src="https://music.163.com/song/media/outer/url?id=5221167.mp3"></audio>
网页嵌入代码如下:  
```html
<audio controls="controls" title="歌名" src="解析地址"></audio>
```
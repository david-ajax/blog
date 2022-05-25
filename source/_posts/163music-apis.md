---
title: 记录一下网易云音乐的开放 API
date: 2022-3-7
---
从网络上获取歌曲信息.
<!--more-->
# 准备源数据
以 Rick Astley 的 Never Gonna Give You Up 为例,  
首先在客户端或网站上获取歌曲的播放地址, 通常是像这样:  
"http://music.163.com/#/m/song?id=5221167"  
根据链接的基础知识, 我们可以把这个 URI 拆分成几个部分:  
Prorocol: http:  
Auth: none  
Hostname(Domain): music.163.com  
Port: none(实际为80)  
Pathname: /#/m/song  
Quary: id=5221167(不带问号)  
我们要用到 Quary 部分  
将"="后的数字分离出来, 下文称为 MusicID  
# 获得歌曲的直接播放链接 
将 MusicID 嵌入解析链接  
格式: https://music.163.com/song/media/outer/url?id=[MusicID].mp3  
比如 http://music.163.com/#/m/song?id=5221167  
就是 https://music.163.com/song/media/outer/url?id=5221167.mp3  
我们还可以将其嵌入到 HTML
效果:  
Never Gonna Give You Up - Rick Astley  
<audio controls="controls" title="Never Gonna Give You Up" src="https://music.163.com/song/media/outer/url?id=5221167.mp3"></audio>  
网页嵌入代码如下:  
```html
<audio controls="controls" title="歌名" src="解析地址"></audio>
```

# 获得歌曲的歌词
将 MusicID 嵌入解析链接  
http://music.163.com/api/song/lyric?os=pc&id=[MusicID]&lv=-1&kv=-1&tv=-1  
得到 JSON 格式的歌词信息  
可以通过去除 "&tv=-1" 来去除翻译  
可以通过去除 "&lv=-1" 来去除原文  
可以通过去除 "&kv=-1" 来去除一些状态信息  
现在, 应该可以做一个不登录的网易云第三方客户端了...  

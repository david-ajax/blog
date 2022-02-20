---
title: 通过 CloudFlare 搭建镜像站(Server Less)
date: 2022-1-13
---
背景:想通过 sourceforge 下载 webmin 的安装包, 然而由于 sf 的服务器位于国外, 下载速度只有不到 10kb/s. 所以想搭建镜像站以实现高速下载. 无奈没有海外服务器, 只能使用 cloudflare workers 凑合.  
<!--more-->
1. 注册并登录一个 cloudflare 账号(略)  
2. 打开[ cloudflare workers 官网](https://workers.cloudflare.com/)
![登录后显示界面](https://i.loli.net/2021/10/06/HkIZlwbrjLeJfcQ.png)
3. 点击"创建 Worker", 进入如下界面
![编辑界面](https://i.loli.net/2021/10/06/sRax8XN1D7fUPvm.png)
4.把原来的代码全部删除, 换成下面代码:
``` javascript
// 目标地址
const upstream = 'www.example.com'

// 代理路径(默认为'/',即根目录)
const upstream_path = '/'

// 目标地址移动版
const upstream_mobile = 'www.example.com'

// 不为以下国家/地区服务
const blocked_region = ['国家代号']

// IP地址黑名单
const blocked_ip_address = ['0.0.0.0', '127.0.0.1']

// 是否使用HTTPS协议
const https = true

// 是否禁用缓存
const disable_cache = false

// 替换网页文件的链接文本(即把目标网站超链接替换成镜像站的超链接)
const replace_dict = {
    '$upstream': '$custom_domain', //将目标网址替换成镜像站网址
    //'增加被替换的链接文本': '替换成的文本'
}

addEventListener('fetch', event => {
    event.respondWith(fetchAndApply(event.request));
})

async function fetchAndApply(request) {
    const region = request.headers.get('cf-ipcountry').toUpperCase();
    const ip_address = request.headers.get('cf-connecting-ip');
    const user_agent = request.headers.get('user-agent');

    let response = null;
    let url = new URL(request.url);
    let url_hostname = url.hostname;

    if (https == true) {
        url.protocol = 'https:';
    } else {
        url.protocol = 'http:';
    }

    if (await device_status(user_agent)) {
        var upstream_domain = upstream;
    } else {
        var upstream_domain = upstream_mobile;
    }

    url.host = upstream_domain;
    if (url.pathname == '/') {
        url.pathname = upstream_path;
    } else {
        url.pathname = upstream_path + url.pathname;
    }

    if (blocked_region.includes(region)) {
        response = new Response('Access denied: WorkersProxy is not available in your region yet.', {
            status: 403
        });
    } else if (blocked_ip_address.includes(ip_address)) {
        response = new Response('Access denied: Your IP address is blocked by WorkersProxy.', {
            status: 403
        });
    } else {
        let method = request.method;
        let request_headers = request.headers;
        let new_request_headers = new Headers(request_headers);

        new_request_headers.set('Host', upstream_domain);
        new_request_headers.set('Referer', url.protocol + '//' + url_hostname);

        let original_response = await fetch(url.href, {
            method: method,
            headers: new_request_headers
        })

        connection_upgrade = new_request_headers.get("Upgrade");
        if (connection_upgrade && connection_upgrade.toLowerCase() == "websocket") {
            return original_response;
        }

        let original_response_clone = original_response.clone();
        let original_text = null;
        let response_headers = original_response.headers;
        let new_response_headers = new Headers(response_headers);
        let status = original_response.status;
		
		if (disable_cache) {
			new_response_headers.set('Cache-Control', 'no-store');
	    }

        new_response_headers.set('access-control-allow-origin', '*');
        new_response_headers.set('access-control-allow-credentials', true);
        new_response_headers.delete('content-security-policy');
        new_response_headers.delete('content-security-policy-report-only');
        new_response_headers.delete('clear-site-data');
		
		if (new_response_headers.get("x-pjax-url")) {
            new_response_headers.set("x-pjax-url", response_headers.get("x-pjax-url").replace("//" + upstream_domain, "//" + url_hostname));
        }
		
        const content_type = new_response_headers.get('content-type');
        if (content_type != null && content_type.includes('text/html') && content_type.includes('UTF-8')) {
            original_text = await replace_response_text(original_response_clone, upstream_domain, url_hostname);
        } else {
            original_text = original_response_clone.body
        }
		
        response = new Response(original_text, {
            status,
            headers: new_response_headers
        })
    }
    return response;
}

async function replace_response_text(response, upstream_domain, host_name) {
    let text = await response.text()

    var i, j;
    for (i in replace_dict) {
        j = replace_dict[i]
        if (i == '$upstream') {
            i = upstream_domain
        } else if (i == '$custom_domain') {
            i = host_name
        }

        if (j == '$upstream') {
            j = upstream_domain
        } else if (j == '$custom_domain') {
            j = host_name
        }

        let re = new RegExp(i, 'g')
        text = text.replace(re, j);
    }
    return text;
}


async function device_status(user_agent_info) {
    var agents = ["Android", "iPhone", "SymbianOS", "Windows Phone", "iPad", "iPod"];
    var flag = true;
    for (var v = 0; v < agents.length; v++) {
        if (user_agent_info.indexOf(agents[v]) > 0) {
            flag = false;
            break;
        }
    }
    return flag;
}
```
5. 点击"保存并部署", 即可通过域名访问镜像站了!

---
title: 让Nginx获取访问者的真实IP
published: 2018-04-23T03:30:00Z
description: ''
updated: 2025-10-09T06:18:24.315Z
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'zh'
abbrlink: 'nginx-real-ip-behind-cloudflare'
---

这项特性需要Nginx模块[ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)来实现

参考[Cloudflare官方文档](https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/)在配置文件的http块中加入

```
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 104.16.0.0/12;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 131.0.72.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2c0f:f248::/32;
set_real_ip_from 2a06:98c0::/29;

# use any of the following two
real_ip_header CF-Connecting-IP;
#real_ip_header X-Forwarded-For;
```

> 未来Hacc的评论：
>
> 嘛，其实就算不这么做后端的应用也能感知到客户端的真实IP，因为可以直接读`CF-Connecting-IP`或者是`X-Forwarded-For`头（如果`X-Forwarded-For`没有被Nginx覆盖掉的话）。但如果我没记错的话，Nginx本身不会把`X-Forwarded-For`所记录的地址视为客户端的真实IP，因此在Nginx日志里看到的地址全都是CDN的地址，`ngx_http_realip_module`模块就是让nginx本身去读所设定的某个头，然后将其视为客户端的IP，然后为了安全起见，可以只信任白名单内地址段传入连接的头，其余情况依旧把IP层的源地址视为客户端地址。
>
> 还是像之前说的那样，如果是Cloudflare的话，[cloudflared](https://github.com/cloudflare/cloudflared)打隧道就好了，`set_real_ip_from`直接填本地回环地址，不需要定期更新Cloudflare的地址段。

---
title: Restoring Original Visitor IPs to Nginx Behind Cloudflare
published: 2018-04-23T03:30:00Z
description: ''
updated: 2025-10-09T06:18:24.315Z
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'en'
abbrlink: 'nginx-real-ip-behind-cloudflare'
---

This feature requires the Nginx module [ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)

Following the [official Cloudflare documentation](https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/), add the following to the `http` block in your configuration file:

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

> A note from Future Hacc:
>
> Well, to be honest, even without this module, the backend application can often detect the client's real IP because it can directly read the `CF-Connecting-IP` or `X-Forwarded-For` headers (assuming `X-Forwarded-For` hasn't been overwritten by Nginx). But, if I remember correctly, Nginx itself won't treat the address in `X-Forwarded-For` as the client's real IP. Therefore, the addresses you see in the Nginx logs will all be from the CDN.
>
> The purpose of the `ngx_http_realip_module` is to make Nginx itself read a specific header you've configured and treat its value as the client's real IP. Then, for security, you can configure it to only trust this header for connections coming from the allow-listed IP ranges (i.e., Cloudflare's IPs). For all other connections, it will fall back to using the original source address from the IP layer as the client's address.
>
> As I've mentioned before, if you're using Cloudflare, it's better to just create a tunnel using `cloudflared`. That way, you can just set `set_real_ip_from` to the local loopback address and no longer need to periodically update the list of Cloudflare's IP ranges.

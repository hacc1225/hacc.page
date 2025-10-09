---
title: 在Vultr上的Archlinux配置IPv6
published: 2018-04-23T02:50:00Z
description: ''
updated: 2025-10-09T05:43:23.522Z
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'zh'
abbrlink: 'vultr-archlinux-ipv6-setup'
---

一直奉行「生命在于折腾」的博主在VPS购买后没多久便将系统换成了Archlinux，配置时为了省事就用了`dhcpcd`自动配置了IP地址。

自动配置的IPv4是完美的，可自动配置的IPv6就出事了。。。之前一直没有用过IPv6做过网站，直到最近脑抽突然想用Cloudflare的IPv6网关时才将问题暴露出来。

服务器可以ping外界IPv6主机，但外界IPv6主机却没法ping服务器。用`systemctl status dhcpcd`查看dhcpcd的日志后发现，这玩意自动获取的IPv6地址和Vultr给的固定IPv6地址不是一个地址。。。

用[man DHCPCD.CONF](https://man.archlinux.org/man/dhcpcd.conf.5)查询，得到dhcpcd的静态地址配置方法。在`/etc/dhcpcd.conf`的尾部加入如下内容

```
interface eth0 #网卡名
static ip6_address= #你的静态IPv6地址
```

然后`sudo systemctl restart dhcpcd`重启`dhcpcd`后就OK了。

---

然而博主在配置Nginx时并没有监听IPv6地址，因此想要使用Cloudflare的IPv6网关还得要在配置文件中的`server`块中有

```
listen [::]:80;
```

或者是

```
listen [::]:443 ssl http2;
```

> 未来Hacc的评论：
>
> 现在的我作为`Systemd`教徒应该就`Systemd`全家桶一把梭了。
>
> 虽然现在已经没法搞清楚当时到底发生了啥导致VPS可以ping外部，但外部没法ping VPS了。但再次发动水晶球占卜，我猜大概应该是`dhcpcd`根据Vultr的RA自己SLAAC配了一个地址，这个地址也是有效的，但它不是Vultr管理面板上所显示的分配给VPS的地址，从而导致了这个问题。

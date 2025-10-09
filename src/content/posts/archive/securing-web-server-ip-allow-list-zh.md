---
title: Web服务IP白名单
published: 2018-04-22T05:33:00Z
description: ''
updated: 2025-10-09T04:33:19+02:00
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'zh'
abbrlink: 'securing-web-server-ip-allow-list'
---

孤陋寡闻的博主某天突然知道还可以通过“全网扫描”这种骚操作来获取CDN背后网站的真实IP。于是乎为了（hao）增（wan）强网站的安全性，博主开始折腾通过白名单来屏蔽全网扫描大法。

博主首先想到的是通过Nginx来屏蔽。

首先通过[https://www.cloudflare.com/ips/](https://www.cloudflare.com/ips/)来get到Cloudflare的IP段。

将`allow`指令添加到配置文件的`http`块下，依次添加Cloudflare的IP段。

```
allow 103.21.244.0/22;
…
allow 2400:cb00::/32;
…
```

然后再添加

```
deny all;
```

访问源站，服务器确实返回了`403`，可这是在https握手之后才返回的。。。

证书暴露了。。。

然后想到让Nginx返回444，说不定可以在握手前断开与白名单外主机的连接。

参考[Ilham Sulaksono的回答](https://serverfault.com/questions/892941/nginx-return-444-on-deny/892952#892952)后，在配置文件的`http`块中加入

```
geo $remote_addr $allowed_trafic {
default false;
103.21.244.0/22 true;
…
2400:cb00::/32 true;
…
}
```

来添加Cloudflare的IP段

在每个`server`块下添加

```
if ( $allowed_trafic = 'false'){
return 444;
}
```

重启Nginx后发现。。。

依然是在握手后才断开连接。。。

好吧，服气。。。

> 未来Hacc的评论：
>
> 当然会这样，Nginx这边在http上屏蔽无论如何都是在会话层的TLS之后了

看来只能用`iptables`屏蔽了。。。

依次输入

```
sudo systemctl enable iptables
sudo systemctl enable ip6tables
```

启用`iptables`

> 未来Hacc的评论：
>
> 其实这里只是启用了加载`iptables`规则的`systemd`服务

然后输入

```
sudo touch /etc/iptables/iptables.rules
sudo touch /etc/iptables/ip6tables.rules
```

新建一个空配置文件

再依次输入

```
sudo systemctl start iptables
sudo systemctl start ip6tables
```

启动`iptables`

> 未来Hacc的评论：
>
> 现在想的话，其实应该`restart`比较合适，因为此时这个服务可能已经启动了，这种情况下刚刚清空`iptables`配置的操作将不会生效。

输入`sudo bash`提权

参考[Frank Rietta的文章](https://rietta.com/blog/2012/09/10/using-iptables-to-require-cloudflare/)还有[Cloudflare的文档](https://support.cloudflare.com/hc/en-us/articles/200169166-How-do-I-whitelist-Cloudflare-s-IP-addresses-in-iptables-)用root权限执行以下指令来将Cloudflare的IP段以及本地回环地址添加进`iptables`。

```
for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -s $i --dport http -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -s $i --dport https -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -s $i --dport http -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -s $i --dport https -j ACCEPT; done
```

然后再输入下列指令来丢弃掉白名单外主机的数据包

```
iptables -A INPUT -p tcp --dport http -j DROP
iptables -A INPUT -p tcp --dport https -j DROP
ip6tables -A INPUT -p tcp --dport http -j DROP
ip6tables -A INPUT -p tcp --dport https -j DROP
```

此时设置就已经完成了，可设置还没有保存，还需要输入

```
iptables-save > /etc/iptables/iptables.rules
ip6tables-save > /etc/iptables/ip6tables.rules
```

将设置保存到配置文件

> 未来Hacc的评论：
>
> 如果是Cloudflare的话，感觉在一般的场景下直接用开源的[cloudflared](https://github.com/cloudflare/cloudflared)打个隧道就好了，这样不仅不用定期更新Cloudflare的IP段，ISP还没法通过SNI读到服务器所托管的网站域名，更隐蔽。

---
title: Securing Web Server with an IP Allow List
published: 2018-04-22T05:33:00Z
description: ''
updated: 2025-10-09T04:33:19+02:00
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'en'
abbrlink: 'securing-web-server-ip-allow-list'
---

I recently learned about a clever trick: using "internet-wide scanning" to discover the real IP address of a website hidden behind a CDN. So, in an effort to enhance my site's security (and for the fun of it), I started tinkering with an allow list to block these scanning methods.

My first thought was to handle this with Nginx.

First, I got the list of Cloudflare's IP ranges from [https://www.cloudflare.com/ips/](https://www.cloudflare.com/ips/)

I added the `allow` directives to the `http` block of my configuration file, adding each of Cloudflare's IP ranges one by one.

```
allow 103.21.244.0/22;
…
allow 2400:cb00::/32;
…
```

And then I added:

```
deny all;
```

When I tried accessing the origin server directly, it did indeed return a 403 Forbidden error, but this only happened after the HTTPS handshake was complete...

My certificate was still exposed...

My next idea was to make Nginx return a `444` error, hoping it might close the connection before the handshake with hosts not on the allow list.

Following a [response by Ilham Sulaksono](https://serverfault.com/questions/892941/nginx-return-444-on-deny/892952#892952), I added the following to the `http` block of my configuration:

```
geo $remote_addr $allowed_trafic {
default false;
103.21.244.0/22 true;
…
2400:cb00::/32 true;
…
}
```

This defined the Cloudflare IP ranges.

Then, under each `server` block, I added:

```
if ( $allowed_trafic = 'false'){
return 444;
}
```

After restarting Nginx, I discovered...

The connection was still being terminated only after the handshake...

Okay, I admit defeat...

> A note from Future Hacc:
>
> Of course, this would happen. Blocking at the HTTP level in Nginx will always occur after the TLS handshake at the session layer.

It seemed like using `iptables` was the only way...

First, I entered these commands to enable iptables:

```
sudo systemctl enable iptables
sudo systemctl enable ip6tables
```

> A note from Future Hacc:
>
> This actually just enables the `systemd` services that load the `iptables` rules on boot.

Then, I entered:

```
sudo touch /etc/iptables/iptables.rules
sudo touch /etc/iptables/ip6tables.rules
```

This creates new, empty configuration files.

Next, I started the services to enable iptables:

```
sudo systemctl start iptables
sudo systemctl start ip6tables
```

> A note from Future Hacc:
>
> Thinking about it now, `restart` would probably be more appropriate here. If the service was already running, the previous step of clearing the rules might not take effect without a restart.

I entered `sudo bash` to get root privileges.

Referencing an [article by Frank Rietta](https://rietta.com/blog/2012/09/10/using-iptables-to-require-cloudflare/) and the [Cloudflare documentation](https://support.cloudflare.com/hc/en-us/articles/200169166-How-do-I-whitelist-Cloudflare-s-IP-addresses-in-iptables-), I executed the following commands as root to add Cloudflare's IP ranges to `iptables`.

```
for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -s $i --dport http -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -s $i --dport https -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -s $i --dport http -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -s $i --dport https -j ACCEPT; done
```

Then, I entered the following commands to drop packets from any host not on the allow list:

```
iptables -A INPUT -p tcp --dport http -j DROP
iptables -A INPUT -p tcp --dport https -j DROP
ip6tables -A INPUT -p tcp --dport http -j DROP
ip6tables -A INPUT -p tcp --dport https -j DROP
```

At this point, the settings were active but not yet saved. To save them, I still needed to enter:

```
iptables-save > /etc/iptables/iptables.rules
ip6tables-save > /etc/iptables/ip6tables.rules
```

This saves the settings to the configuration files.

> A note from Future Hacc:
>
> For Cloudflare specifically, it's probably easier in most cases to just use the open-source [cloudflared](https://github.com/cloudflare/cloudflared) to create a tunnel. Not only does this save you from having to periodically update Cloudflare's IP list, but your ISP also can't read the hosted domain names via SNI, making it more private.

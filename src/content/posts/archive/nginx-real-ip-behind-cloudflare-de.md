---
title: Echte Besucher-IPs für Nginx hinter Cloudflare wiederherstellen
published: 2018-04-23T03:30:00Z
description: ''
updated: 2025-10-09T06:18:24.315Z
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'de'
abbrlink: 'nginx-real-ip-behind-cloudflare'
---

Diese Funktion erfordert das Nginx-Modul [ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)

Laut der [offiziellen Cloudflare-Dokumentation](https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/) fügen Sie Folgendes zum `http`-Block Ihrer Konfigurationsdatei hinzu:

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

> Eine Anmerkung von meinem Zukunfts-Ich:
>
> Ehrlich gesagt kann die Backend-Anwendung auch ohne dieses Modul oft die echte IP des Clients erkennen, da sie die Header `CF-Connecting-IP` oder `X-Forwarded-For` direkt auslesen kann (vorausgesetzt, `X-Forwarded-For` wurde nicht von Nginx überschrieben). Aber, wenn ich mich recht erinnere, behandelt Nginx selbst die Adresse im `X-Forwarded-For`-Header nicht als die echte IP des Clients. Daher sind die Adressen, die Sie in den Nginx-Logs sehen, allesamt Adressen des CDN.
>
> Der Zweck des `ngx_http_realip_module` ist es, Nginx selbst dazu zu bringen, einen von Ihnen konfigurierten, spezifischen Header zu lesen und dessen Wert als die echte IP des Clients zu behandeln. Aus Sicherheitsgründen können Sie es dann so konfigurieren, dass es diesem Header nur bei Verbindungen vertraut, die von den IP-Bereichen der Allow-List stammen (d.h. den IPs von Cloudflare). Bei allen anderen Verbindungen greift es darauf zurück, die ursprüngliche Quelladresse von der IP-Schicht als Adresse des Clients zu verwenden.
>
> Wie ich bereits erwähnt habe, ist es bei Verwendung von Cloudflare besser, einfach mit `cloudflared` einen Tunnel zu erstellen. Auf diese Weise kann man `set_real_ip_from` einfach auf die lokale Loopback-Adresse setzen und muss die Liste der Cloudflare-IP-Bereiche nicht mehr regelmäßig aktualisieren.

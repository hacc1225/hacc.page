---
title: IPv6 für Arch Linux auf Vultr einrichten
published: 2018-04-23T02:50:00Z
description: ''
updated: 2025-10-09T05:43:23.522Z
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'de'
abbrlink: 'vultr-archlinux-ipv6-setup'
---

Als überzeugter Anhänger des Mottos „Leben bedeutet Herumbasteln“ habe ich das Betriebssystem auf meinem VPS kurz nach dem Kauf auf Arch Linux umgestellt. Aus Bequemlichkeit habe ich bei der Einrichtung einfach `dhcpcd` für die automatische Konfiguration der IP-Adresse genutzt.

Die automatische Konfiguration für IPv4 funktionierte einwandfrei, aber bei IPv6 stieß ich auf ein Problem. Ich hatte zuvor noch nie eine Webseite über IPv6 gehostet, daher trat das Problem erst zutage, als ich kürzlich aus einer Laune heraus das IPv6-Gateway von Cloudflare nutzen wollte.

Der Server konnte externe IPv6-Hosts anpingen, aber externe Hosts konnten den Server nicht erreichen. Nachdem ich die `dhcpcd`-Logs mit `systemctl status dhcpcd` überprüft hatte, entdeckte ich das Problem: Die automatisch bezogene IPv6-Adresse war nicht dieselbe wie die von Vultr bereitgestellte statische IPv6-Adresse.

Ich habe die [Manpage für `dhcpcd.conf`](https://man.archlinux.org/man/dhcpcd.conf.5) nachgeschlagen und die Methode zur Konfiguration einer statischen Adresse gefunden. Ich musste lediglich Folgendes am Ende der `/etc/dhcpcd.conf` hinzufügen:

```
interface eth0 #Name deiner Netzwerkschnittstelle
static ip6_address= #Deine statische IPv6-Adresse
```

Ein kurzer Neustart mit `sudo systemctl restart dhcpcd` und alles funktionierte.

---

Allerdings gab es noch einen weiteren Schritt. Bei der ursprünglichen Nginx-Einrichtung hatte ich nicht konfiguriert, dass es auf der IPv6-Adresse lauscht. Um das IPv6-Gateway von Cloudflare nutzen zu können, benötigte ich also zusätzlich eine der folgenden Zeilen in meinem `server`-Block:

```
listen [::]:80;
```

oder

```
listen [::]:443 ssl http2;
```

> Eine Anmerkung von meinem Zukunfts-Ich:
>
> Als `systemd`-Jünger würde ich heutzutage wahrscheinlich voll und ganz auf die komplette `systemd`-Suite setzen.
>
> Obwohl es heute unmöglich ist, genau herauszufinden, was damals zu der Situation führte, dass der VPS externe Hosts anpingen konnte, aber nicht umgekehrt, wäre meine Vermutung diese: `dhcpcd` hat wahrscheinlich die RA von Vultr genutzt, um automatisch eine Adresse per SLAAC zu konfigurieren. Obwohl diese Adresse gültig war, war es nicht die primäre statische Adresse, die dem VPS im Vultr-Control-Panel zugewiesen wurde, was zu dem Problem mit eingehenden Verbindungen führte.

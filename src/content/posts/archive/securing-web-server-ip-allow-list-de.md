---
title: Absicherung eines Webservers mit einer IP-Allow-List
published: 2018-04-22T05:33:00Z
description: ''
updated: 2025-10-09T04:33:19+02:00
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'de'
abbrlink: 'securing-web-server-ip-allow-list'
---

Ich habe kürzlich von einem cleveren Trick erfahren: Wie man durch „netzwerkweite Scans“ die echte IP-Adresse einer Webseite aufdeckt, die hinter einem CDN versteckt ist. Um also die Sicherheit meiner Seite zu erhöhen (und einfach zum Spaß), habe ich angefangen, mit einer Allow-List herumzuexperimentieren, um diese Scan-Methoden zu blockieren.

Mein erster Gedanke war, das Ganze mit Nginx zu lösen.

Zuerst habe ich mir die Liste der IP-Bereiche von Cloudflare von [https://www.cloudflare.com/ips/](https://www.cloudflare.com/ips/) geholt.

Ich habe die `allow`-Anweisungen in den `http`-Block meiner Konfigurationsdatei eingefügt und jeden IP-Bereich von Cloudflare einzeln hinzugefügt.

```
allow 103.21.244.0/22;
…
allow 2400:cb00::/32;
…
```

Und dann habe ich hinzugefügt:

```
deny all;
```

Als ich versuchte, direkt auf den Ursprungsserver (Origin Server) zuzugreifen, gab dieser zwar tatsächlich einen 403-Forbidden-Fehler zurück, aber dies geschah erst, nachdem der HTTPS-Handshake abgeschlossen war...

Mein Zertifikat war also immer noch einsehbar...

Meine nächste Idee war, Nginx einen `444`-Fehler zurückgeben zu lassen, in der Hoffnung, dass die Verbindung zu Hosts, die nicht auf der Allow-List stehen, so vor dem Handshake beendet würde.

Nach einer [Antwort von Ilham Sulaksono](https://serverfault.com/questions/892941/nginx-return-444-on-deny/892952#892952) habe ich Folgendes in den `http`-Block meiner Konfiguration eingefügt:

```
geo $remote_addr $allowed_trafic {
default false;
103.21.244.0/22 true;
…
2400:cb00::/32 true;
…
}
```

Dadurch wurden die IP-Bereiche von Cloudflare definiert.

Anschließend habe ich unter jedem `server`-Block Folgendes hinzugefügt:

```
if ( $allowed_trafic = 'false'){
return 444;
}
```

Nach einem Neustart von Nginx stellte ich fest...

Die Verbindung wurde immer noch erst nach dem Handshake beendet...

Okay, ich gebe mich geschlagen...

> Eine Anmerkung von meinem Zukunfts-Ich:
>
> Natürlich würde das passieren. Eine Blockade auf der HTTP-Ebene in Nginx findet zwangsläufig immer erst nach dem TLS-Handshake auf der Sitzungsschicht (Session Layer) statt.

Es schien, als wäre der Einsatz von `iptables` der einzige Weg...

Zuerst habe ich diese Befehle eingegeben, um `iptables` zu aktivieren:

```
sudo systemctl enable iptables
sudo systemctl enable ip6tables
```

> Eine Anmerkung von meinem Zukunfts-Ich:
>
> Tatsächlich aktiviert dies nur die `systemd`-Dienste, die die `iptables`-Regeln beim Systemstart laden.

Dann habe ich eingegeben:

```
sudo touch /etc/iptables/iptables.rules
sudo touch /etc/iptables/ip6tables.rules
```

Dies erstellt neue, leere Konfigurationsdateien.

Als Nächstes habe ich die Dienste gestartet:

```
sudo systemctl start iptables
sudo systemctl start ip6tables
```

> Eine Anmerkung von meinem Zukunfts-Ich:
>
> Wenn ich jetzt darüber nachdenke, wäre `restart` hier wahrscheinlich besser geeignet. Wenn der Dienst bereits lief, würde der vorherige Schritt des Löschens der Regeln ohne einen Neustart möglicherweise nicht wirksam werden.

Ich habe `sudo bash` eingegeben, um Root-Rechte zu erhalten.

Unter Bezugnahme auf einen [Artikel von Frank Rietta](https://rietta.com/blog/2012/09/10/using-iptables-to-require-cloudflare/) und die [Cloudflare-Dokumentation](https://support.cloudflare.com/hc/en-us/articles/200169166-How-do-I-whitelist-Cloudflare-s-IP-addresses-in-iptables-) habe ich die folgenden Befehle als Root ausgeführt, um die IP-Bereiche von Cloudflare zu `iptables` hinzuzufügen.

```
for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -s $i --dport http -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -s $i --dport https -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -s $i --dport http -j ACCEPT; done
for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -s $i --dport https -j ACCEPT; done
```

Dann habe ich die folgenden Befehle eingegeben, um Pakete von allen Hosts zu verwerfen, die nicht auf der Allow-List stehen:

```
iptables -A INPUT -p tcp --dport http -j DROP
iptables -A INPUT -p tcp --dport https -j DROP
ip6tables -A INPUT -p tcp --dport http -j DROP
ip6tables -A INPUT -p tcp --dport https -j DROP
```

An diesem Punkt waren die Einstellungen aktiv, aber noch nicht gespeichert. Um sie zu speichern, musste ich noch Folgendes eingeben:

```
iptables-save > /etc/iptables/iptables.rules
ip6tables-save > /etc/iptables/ip6tables.rules
```

Dies speichert die Einstellungen in den Konfigurationsdateien.

> Eine Anmerkung von meinem Zukunfts-Ich:
>
> Speziell für Cloudflare ist es in den meisten Fällen wahrscheinlich einfacher, einfach das Open-Source-Tool [cloudflared](https://github.com/cloudflare/cloudflared) zu verwenden, um einen Tunnel zu erstellen. Das erspart einem nicht nur die regelmäßige Aktualisierung der Cloudflare-IP-Liste, sondern der eigene ISP kann auch die gehosteten Domainnamen nicht mehr per SNI auslesen, was die Privatsphäre erhöht.

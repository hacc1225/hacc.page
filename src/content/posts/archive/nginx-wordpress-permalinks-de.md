---
title: Permalinks für WordPress in Nginx konfigurieren
published: 2018-04-21T18:49:00Z
description: ''
updated: ''
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'de'
abbrlink: 'nginx-wordpress-permalinks'
---

Laut der [offiziellen Nginx-Dokumentation](https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/) und der [offiziellen WordPress-Dokumentation](https://codex.wordpress.org/Nginx) fügen Sie die folgende Zeile in Ihren `location`-Block ein:
```
try_files $uri $uri/ /index.php?$args;
```

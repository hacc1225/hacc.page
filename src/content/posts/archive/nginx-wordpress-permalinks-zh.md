---
title: Nginx的Wordpress伪静态配置
published: 2018-04-21T18:49:00Z
description: ''
updated: ''
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'zh'
abbrlink: 'nginx-wordpress-permalinks'
---

参考[Nginx官方文档](https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/)和[Wordpress官方文档](https://codex.wordpress.org/Nginx)在location块中加入
```
try_files $uri $uri/ /index.php?$args;
```

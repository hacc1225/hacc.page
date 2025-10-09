---
title: Configuring Permalinks for WordPress on Nginx
published: 2018-04-21T18:49:00Z
description: ''
updated: ''
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'en'
abbrlink: 'nginx-wordpress-permalinks'
---

According to the [official Nginx documentation](https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/) and the [official WordPress documentation](https://codex.wordpress.org/Nginx), add the following line inside your `location` block:
```
try_files $uri $uri/ /index.php?$args;
```

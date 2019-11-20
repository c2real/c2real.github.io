---
layout: post
title:  "Setup TTRSS with self-host and nginx reverse proxy "
date:   2019-11-15 9:10:00 +0800
categories: TTRSS Ngnix
---

### About TTRSS

为了解决信息获取以及信息过滤的问题，我需要一个自定义RSS源，那么TTRSS(Tiny Tiny RSS is a free and open source web-based news feed (RSS/Atom) reader and aggregator)则成了目前最佳的选择。

有了TTRSS，就可以从混乱的微博时间线，社交媒体的广告摆脱出来，其实是减轻信息焦虑的好办法。摆脱原有这些媒体的客户端之后，就会发现每天信息源更新是有限的，而社交媒体为了最大化获取用户时间，则会刻意用各种手段让用户沉浸在客户端中，制造一种无限信息更新的假象。

搭建TTRSS，需要VPS主机支持，可以自行配置数据库，PHP等运行环境，也可以用Docker，来一次性解决所有问题。

[Awsome-TTRSS]: https://github.com/HenryQW/Awesome-TTRSS

借用开发者的话

Awesome TTRSS aims to provide a powerful **Dockerized all-in-one** solution for [Tiny Tiny RSS](https://tt-rss.org/), an open source RSS feed reader and aggregator written in PHP, with enhanced user experience via simplified deployment and a list of curated plugins. 



# Nginx Reverse Proxy

为了保证数据安全，现在https已经是大势所趋。通过NameCheap 和Let's Encrypt可以低成本的获得一个域名和SSL证书。与网站绑定后，看见浏览器地址栏的锁，会更加安心。

如果打算使用子目录来访问TTRSS时，例如https://www.test.com/ttrss/, 可以使用Nginx的反向代理，把对网站子目录ttrss的访问映射到内部地址上，例如http://127.0.0.1:8080/。

需要注意的是，由于TTRSS对变量SELF_URL_PATH的校验，如果内部地址的子目录路径和外部地址的子路路径不一致，则会出现SELF_URL_PATH错误。

那么，为了解决此问题，反向代理可以这样配置

```nginx
location /ttrss/ {
        proxy_redirect off;
        proxy_pass http://localhost:8080/ttrss/;

        proxy_set_header  Host                $http_host;
        proxy_set_header  X-Real-IP           $remote_addr;
        proxy_set_header  X-Forwarded-Ssl     on;
        proxy_set_header  X-Forwarded-For     $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto   $scheme;
        proxy_set_header  X-Frame-Options     SAMEORIGIN;

        }
```

同时，需要把原Docker镜像中的/var/www/目录下所有内容移动到/var/www/ttrss/目录下，并且更新update守护进程的配置命令。即 s6/update-daemon/run 这个文件

```sh
#!/bin/sh

exec s6-setuidgid nobody php /var/www/ttrss/update_daemon2.php
```

因为每次容器启动时，都要启动这个守护进程。

最后需要修改 src/configure-db.php这个文件，如果容器已经存在，则可以通过 

```sh
docker exec -it ttrss sh
```

进入容器，在其内部修改

```php

$confpath = '/var/www/ttrss/config.php';
$config = array();

```

Enjoy it!


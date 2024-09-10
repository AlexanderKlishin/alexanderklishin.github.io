---
layout: page
title: Nginx
---

# Doc

<https://www.freecodecamp.org/news/the-nginx-handbook/>

## **http_rewrite**

<https://nginx.org/en/docs/http/ngx_http_rewrite_module.html>

- break
- if
- return
- rewrite
- rewrite_log
- set
- uninitialized_variable_warn

```bash
Syntax:	return code [text];
        return code URL;
        return URL;
```

## http_proxy

<https://nginx.org/ru/docs/http/ngx_http_proxy_module.html>

## http_slice

<https://nginx.org/en/docs/http/ngx_http_slice_module.html>

<https://blog.nginx.org/blog/smart-efficient-byte-range-caching-nginx>

# Doc dev

<https://nginx.org/en/docs/dev/development_guide.html>

- Phases: <https://nginx.org/en/docs/dev/development_guide.html#http_phases>

## module

<https://www.evanmiller.org/nginx-modules-guide.html>

<https://www.evanmiller.org/nginx-modules-guide-advanced.html>

<http://www.grid.net.ru/nginx/nginx-modules.html>

### var compile

<https://stackoverflow.com/questions/43288246/whats-the-best-way-for-nginx-modules-to-expand-variables-in-directives>

### hello world

<https://tejgop.github.io/nginx-module-guide/#introduction>


# Run

## From src

```bash
mkdir -p /tmp/logs/
./nginx/objs/nginx -p /tmp/ -c /path-to/nginx.conf
```

## gdb

```bash
gdb --arg  ./nginx/objs/nginx -p /tmp -c /path-to/nginx.conf
```

## asan

```bash
ASAN_OPTIONS=detect_leaks=1:log_path=/tmp/asan_nginx
```

## core

```bash
ulimit -c unlimited
echo "/tmp/core.%e.%p" | sudo tee /proc/sys/kernel/core_pattern
cat /proc/sys/kernel/core_pattern

```

# Test

## **Test::Nginx**

### doc

<https://github.com/openresty/test-nginx>

<https://openresty.gitbooks.io/programming-openresty/content/testing/index.html>

# config

```bash
pid /tmp/logs/nginx.pid;
worker_processes 1;
daemon off;
master_process off;

error_log /dev/stdout debug;

http {

    server {
        client_body_temp_path /tmp/logs/client_body;
        proxy_temp_path /tmp/logs/proxy;
        fastcgi_temp_path /tmp/logs/fastcgi;
        uwsgi_temp_path /tmp/logs/uwsgi;
        scgi_temp_path /tmp/logs/scgi;
        vkupload_path /tmp/logs/vkupload;

        listen 127.0.0.1:8081 reuseport;
				access_log /dev/stdout;

        location / {
            return 200 "ua: $http_user_agent\n";
        }
    }
}

events {
    accept_mutex off;
    worker_connections 1024;
}
```

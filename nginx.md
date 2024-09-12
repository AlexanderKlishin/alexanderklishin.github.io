---
layout: page
title: Nginx
---

# Doc

<https://www.freecodecamp.org/news/the-nginx-handbook/>

## http_rewrite

<https://nginx.org/en/docs/http/ngx_http_rewrite_module.html>

### directives
- break
- if
- return
- rewrite
  <https://blog.nginx.org/blog/creating-nginx-rewrite-rules>
- rewrite_log
- set
- uninitialized_variable_warn

```bash
Syntax:	return code [text];
        return code URL;
        return URL;
```

## http_headers

https://nginx.org/en/docs/http/ngx_http_headers_module.html

### directives

- add_header
- add_trailer
- expires

## http_proxy

<https://nginx.org/ru/docs/http/ngx_http_proxy_module.html>

## http_slice

<https://nginx.org/en/docs/http/ngx_http_slice_module.html>

<https://blog.nginx.org/blog/smart-efficient-byte-range-caching-nginx>

# Doc dev

<https://nginx.org/en/docs/dev/development_guide.html>

## phases

<https://nginx.org/en/docs/dev/development_guide.html#http_phases>

- NGX_HTTP_POST_READ_PHASE
- NGX_HTTP_SERVER_REWRITE_PHASE
- NGX_HTTP_FIND_CONFIG_PHASE
- NGX_HTTP_REWRITE_PHASE
- NGX_HTTP_POST_REWRITE_PHASE
- NGX_HTTP_PREACCESS_PHASE
- NGX_HTTP_ACCESS_PHASE

    <https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy>
    > The client must pass handlers registered at this phase.
    > Satisfy directive permits to continue if any handlers authorizes the client.

- NGX_HTTP_POST_ACCESS_PHASE
- NGX_HTTP_PRECONTENT_PHASE

    > modules: try_files mirror

- NGX_HTTP_CONTENT_PHASE

    > modules called sequentially until one of them produces the output

- NGX_HTTP_LOG_PHASE

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

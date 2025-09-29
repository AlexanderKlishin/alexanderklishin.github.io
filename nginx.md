---
layout: page
title: Nginx
---

# Doc

<https://www.freecodecamp.org/news/the-nginx-handbook/>

## vars

<https://nginx.org/en/docs/varindex.html>

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

<https://nginx.org/en/docs/http/ngx_http_headers_module.html>

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

    > handler: any return code other than NGX_OK, NGX_DECLINED, NGX_AGAIN, NGX_DONE is considered a denial

    <https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy>
    > The client must pass handlers registered at this phase.
    > Satisfy directive permits to continue if any handlers authorizes the client.
- NGX_HTTP_POST_ACCESS_PHASE
- NGX_HTTP_PRECONTENT_PHASE

    > modules: try_files mirror
- NGX_HTTP_CONTENT_PHASE

    > modules called sequentially until one of them produces the output
- NGX_HTTP_LOG_PHASE

### handler return codes

- NGX_OK

    Proceed to the next phase.
- NGX_DECLINED

    Proceed to the next handler of the current phase.
    If the current handler is the last in the current phase, move to the next phase.
- NGX_AGAIN, NGX_DONE

    Suspend phase handling until some future event which can be an asynchronous I/O operation or just a delay, for example. It is assumed, that phase handling will be resumed later by calling ngx_http_core_run_phases().
- other

    Any other value returned by the phase handler is treated as a request finalization code, in particular, an HTTP response code. The request is finalized with the code provided.

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

# init

```C
/* nginx/src/core/nginx.c */
int main(int argc, char *const *argv) {
    cycle = ngx_init_cycle(&init_cycle);

    if (ngx_process == NGX_PROCESS_SINGLE) {
        ngx_single_process_cycle(cycle);
    } else {
        ngx_master_process_cycle(cycle);
    }
}

/* nginx/src/core/ngx_cycle.c */
ngx_cycle_t *ngx_init_cycle(ngx_cycle_t *old_cycle) {
    ...
    for (i = 0; cycle->modules[i]; i++) {
        if (cycle->modules[i]->type != NGX_CORE_MODULE) {
            continue;
        }
        module->create_conf(cycle) /* create core module conf */
    }

    ngx_conf_parse(&conf, &cycle->conf_file)

    for (i = 0; cycle->modules[i]; i++) {
        if (cycle->modules[i]->type != NGX_CORE_MODULE) {
            continue;
        }
        module->init_conf(cycle) /* init core module cond */
    }

    /* create shared memory */
    for (i = 0; /* void */ ; i++) {
        ngx_shm_alloc(&shm_zone[i].shm)
        shm_zone[i].init(&shm_zone[i], data)
    }

    ngx_init_modules(cycle) /* cycle->modules[i]->init_module(cycle) */

    /* free old shared memory */
    for (i = 0; /* void */ ; i++) {
        ngx_shm_free(&oshm_zone[i].shm);
    }
}

/* nginx/src/os/unix/ngx_process_cycle.c */
void ngx_master_process_cycle(ngx_cycle_t *cycle) {
    ngx_start_worker_processes(cycle, ccf->worker_processes, NGX_PROCESS_RESPAWN);
}

void ngx_start_worker_processes(ngx_cycle_t *cycle, ngx_int_t n, ngx_int_t type) {
    for (i = 0; i < n; i++) {
        ngx_spawn_process(cycle, ngx_worker_process_cycle, ...);
    }
}

void ngx_worker_process_cycle(ngx_cycle_t *cycle, void *data) {
        ngx_worker_process_init(cycle, worker);
}

void ngx_worker_process_init(ngx_cycle_t *cycle, ngx_int_t worker) {

    if (geteuid() == 0) {
        setgid(ccf->group)
        setuid(ccf->user)
    }

    if (ccf->working_directory.len) {
        chdir((char *) ccf->working_directory.data)
    }

    for (i = 0; cycle->modules[i]; i++) {
        cycle->modules[i]->init_process(cycle)
    }
}

```

# init http
```
/* nginx/src/http/ngx_http.c */
static ngx_command_t  ngx_http_commands[] = {
    { ngx_string("http"), NGX_MAIN_CONF|NGX_CONF_BLOCK|NGX_CONF_NOARGS,
      ngx_http_block, 0, 0, NULL },

static char *ngx_http_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    ctx->main_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    for (m = 0; cf->cycle->modules[m]; m++) {
        if (cf->cycle->modules[m]->type != NGX_HTTP_MODULE) { continue; }
        ctx->main_conf[mi] = module->create_main_conf(cf);
        ctx->srv_conf[mi] = module->create_srv_conf(cf);
        ctx->loc_conf[mi] = module->create_loc_conf(cf);
    }

    for (m = 0; cf->cycle->modules[m]; m++) {
        if (cf->cycle->modules[m]->type != NGX_HTTP_MODULE) { continue; }
        module->preconfiguration(cf)
    }

    rv = ngx_conf_parse(cf, NULL);

    for (m = 0; cf->cycle->modules[m]; m++) {
        if (cf->cycle->modules[m]->type != NGX_HTTP_MODULE) { continue; }

        module->init_main_conf(cf, ctx->main_conf[mi]);

        /* !!! merge servers, location conf */
        ngx_http_merge_servers(cf, cmcf, module, mi);
    }

    for (m = 0; cf->cycle->modules[m]; m++) {
        if (cf->cycle->modules[m]->type != NGX_HTTP_MODULE) { continue; }

        module->postconfiguration(cf)
    }

}

/* nginx/src/http/ngx_http_core_module.c */
static ngx_command_t  ngx_http_core_commands[] = {
    { ngx_string("server"), NGX_HTTP_MAIN_CONF|NGX_CONF_BLOCK|NGX_CONF_NOARGS,
      ngx_http_core_server, 0, 0, NULL },

    { ngx_string("location"),
      NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_CONF_BLOCK|NGX_CONF_TAKE12,
      ngx_http_core_location, NGX_HTTP_SRV_CONF_OFFSET, 0, NULL },

    { ngx_string("limit_except"), NGX_HTTP_LOC_CONF|NGX_CONF_BLOCK|NGX_CONF_1MORE,
      ngx_http_core_limit_except, NGX_HTTP_LOC_CONF_OFFSET, 0, NULL },

char * ngx_http_core_server(ngx_conf_t *cf, ngx_command_t *cmd, void *dummy) {
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    ctx->main_conf = http_ctx->main_conf;
    ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    for (i = 0; cf->cycle->modules[i]; i++) {
        if (cf->cycle->modules[i]->type != NGX_HTTP_MODULE) { continue; }
        module->create_srv_conf(cf);
        module->create_loc_conf(cf);
    }

    rv = ngx_conf_parse(cf, NULL);
}

char * ngx_http_core_location(ngx_conf_t *cf, ngx_command_t *cmd, void *dummy) {
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    pctx = cf->ctx;
    ctx->main_conf = pctx->main_conf;
    ctx->srv_conf = pctx->srv_conf;
    ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    for (i = 0; cf->cycle->modules[i]; i++) {
        if (cf->cycle->modules[i]->type != NGX_HTTP_MODULE) { continue; }
        module->create_loc_conf(cf);
    }

    rv = ngx_conf_parse(cf, NULL);
}

char * ngx_http_core_limit_except(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    pctx = cf->ctx;
    ctx->main_conf = pctx->main_conf;
    ctx->srv_conf = pctx->srv_conf;

    for (i = 0; cf->cycle->modules[i]; i++) {
        if (cf->cycle->modules[i]->type != NGX_HTTP_MODULE) { continue; }
        module->create_loc_conf(cf);
    }
}
```

# http client

```C
/* nginx/src/http/ngx_http_request.c */

void ngx_http_init_connection(ngx_connection_t *c) {
    rev = c->read;
    rev->handler = ngx_http_wait_request_handler;
    c->write->handler = ngx_http_empty_handler;
}

void ngx_http_wait_request_handler(ngx_event_t *rev) {
    c->data = ngx_http_create_request(c);

    rev->handler = ngx_http_process_request_line;
    ngx_http_process_request_line(rev);
}

void ngx_http_process_request_line(ngx_event_t *rev) {
            rev->handler = ngx_http_process_request_headers;
            ngx_http_process_request_headers(rev);
}

void ngx_http_process_request_headers(ngx_event_t *rev) {
            ngx_http_process_request(r);
}

void ngx_http_process_request(ngx_http_request_t *r) {
    c->read->handler = ngx_http_request_handler;
    c->write->handler = ngx_http_request_handler;
    r->read_event_handler = ngx_http_block_reading;

    ngx_http_handler(r);
}

/* nginx/src/http/ngx_http_core_module.c */

void ngx_http_handler(ngx_http_request_t *r) {
    r->write_event_handler = ngx_http_core_run_phases;
    ngx_http_core_run_phases(r);
}
```

# http upstream

```C
/* nginx/src/http/modules/ngx_http_proxy_module.c */

static char *ngx_http_proxy_pass(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
    if (plcf->upstream.upstream || plcf->proxy_lengths) {
        return "is duplicate";
    }

    if (n) {
            ngx_http_script_compile(&sc)
        return NGX_CONF_OK;
    }

    u.url.len = url->len - add;
    u.url.data = url->data + add;
    plcf->upstream.upstream = ngx_http_upstream_add(cf, &u, 0);
}

static ngx_int_t ngx_http_proxy_handler(ngx_http_request_t *r) {
    ngx_http_upstream_t         *u;

    ngx_http_upstream_create(r)

    ctx = ngx_pcalloc(r->pool, sizeof(ngx_http_proxy_ctx_t));
    ngx_http_set_ctx(r, ctx, ngx_http_proxy_module);

    u = r->upstream;

    if (plcf->proxy_lengths == NULL) {
        ctx->vars = plcf->vars;
    } else {
            ngx_http_proxy_eval(r, ctx, plcf)

    u->conf = &plcf->upstream;

    ngx_http_read_client_request_body(r, ngx_http_upstream_init)

}
```

```C
/* nginx/src/http/ngx_http_upstream.h */
typedef struct {

    ngx_uint_t                       next_upstream;
} ngx_http_upstream_conf_t;
```

```C
/* nginx/src/http/ngx_http_upstream.c */

void ngx_http_upstream_init(ngx_http_request_t *r) {
}

static void ngx_http_upstream_init_request(ngx_http_request_t *r) {

        uscf = ngx_http_upstream_rbtree_lookup(umcf, host);

    uscf->peer.init(r, uscf)

}

static void
ngx_http_upstream_process_header(ngx_http_request_t *r, ngx_http_upstream_t *u) {

        ngx_http_upstream_next(r, u, NGX_HTTP_UPSTREAM_FT_TIMEOUT);

        ngx_http_upstream_next(r, u, NGX_HTTP_UPSTREAM_FT_ERROR);

            ngx_http_upstream_next(r, u, NGX_HTTP_UPSTREAM_FT_ERROR);

        ngx_http_upstream_next(r, u, NGX_HTTP_UPSTREAM_FT_INVALID_HEADER);

    if (u->headers_in.status_n >= NGX_HTTP_SPECIAL_RESPONSE) {
        ngx_http_upstream_test_next(r, u)

}

static ngx_int_t
ngx_http_upstream_test_next(ngx_http_request_t *r, ngx_http_upstream_t *u) {

            ngx_http_upstream_next(r, u, un->mask);

}

static void
ngx_http_upstream_next(ngx_http_request_t *r, ngx_http_upstream_t *u,
                       ngx_uint_t ft_type) {

    ngx_http_upstream_connect(r, u);
}

static void
ngx_http_upstream_connect(ngx_http_request_t *r, ngx_http_upstream_t *u)
{
    rc = ngx_event_connect_peer(&u->peer);
    /* {
        rc = pc->get(pc, pc->data);
        if (rc != NGX_OK) {
            return rc;
    }*/

    }

}

```

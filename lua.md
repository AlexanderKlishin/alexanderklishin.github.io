---
layout: page
title: lua
<!--permalink: /lua/-->
---

# version diff

| feature | JIT | 5.1 | 5.2 | 5.3 |
|---------|-----|-----|-----|-----|
| C lua_yeild inside 'for' | ok | ERROR: cannot yield across C bounary | ok | ok |
| C lua_resume(lua_State \*L, ... ) params | int narg | int narg | lua_State \*from, int nargs | lua_State \*from, int nargs |
| unpack | unpack | unpack | table.unpack | table.unpack |

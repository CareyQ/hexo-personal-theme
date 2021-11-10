---
title: Nginx 禁止未绑定域名访问
categories: 博客搭建
date: 2020-03-13
url: blog-004
---

今天突然发现一个域名访问后和我的博客完全一样，让我惊讶了一把，于是和小伙伴进行交流，最终通过修改 Nginx 配置后，那个域名不能再访问了，十分感谢小伙伴 @[MonoLogueChi](https://blog.xxwhite.com/)。

Nginx 在开始处理一个 http 请求时，会取出 header 头中的 host，与「nginx.conf」配置中的每个 server 块的「**server_name**」进行匹配，然后决定由哪一个 server 块来处理请求。

**server_name 与 host 匹配的优先级**

1. 完全匹配，如果找到多个，则使用第一个

```nginx
server {
    listen       80;
    server_name: staunchkai.com;
    ...
}
```

2. 以通配符 \* 开头的字符串，如果找到多个，则使用最长的

```nginx
server {
    listen       80;
    server_name *.staunchkai.com;
    ...
}
```

3. 以通配符 \* 结尾的字符串，如果找到多个，则使用最长的

```nginx
server {
    listen       80;
    server_name: staunchkai.*;
    ...
}
```

4. 正则匹配（以 ~ 开头），如果找到多个，则使用第一个

```nginx
server {
    listen       80;
    server_name: ~^(?.+)\.staunchkai\.com$;
    ...
}
```

5. 以上都没有，则选择一个 listen 字段带有 `default server` 字符串的块

```nginx
server {
    listen       80 default_server;
    server_name: _;
    return 444;
}
```

6. 前五条都没有，则匹配 listen 端口的第一个 server 块

知道上述匹配规则，就可以对没有绑定的域名进行处理了。

**Nginx 禁止未绑定域名访问**

```nginx
# 只使用了 80 端口
server {
    listen       80 default_server;
    server_name: _;
    return 444;
}
```

**通过返回 444 这个 nginx 的非标准错误码断开与浏览器的链接**

若是同时使用 80 和 443 两个端口，则配置如下：

```nginx
server {
    listen       80 default_server;
    listen       443 ssl default_server;
    server_name: _;
    ssl_certificate <path to cert>;
    ssl_certificate_key <path to key>;
    return 444;
}
```

配置完成后使用命令 `nginx -t` 检查，`nginx -s reload` 重新加载即可。

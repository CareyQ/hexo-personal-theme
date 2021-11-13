---
title: Hexo 博客开启 HTTPS
categories: 博客搭建
tag:
    - Hexo
    - 腾讯云
    - SSL
date: 2018-09-29
url: blog-002
---

我的服务器是腾讯云的，域名是 ~~阿里云~~（2020 年 9 月更换为腾讯云）的，这篇文章用于记录 Hexo 博客开启 `https` 的过程，网站具备 SSL 在目前差不多算是标配了。

## 获取证书

证书选择的是腾讯云 **免费版 DV**，有效期为一年，腾讯云的证书申请流程，[点击此处](https://cloud.tencent.com/document/product/400/6813)进行查看，在验证域名所有权审核通过后，就可以将解析记录删除掉，到[证书管理](https://console.cloud.tencent.com/ssl)处下载证书。

下载的证书解压出来后的结构应该如下：

- ~~staunchkai.com~~
- careyq.cool
  - Apache
  - IIS
  - Nginx
  - Tomcat
  - ~~staunchkai.com.csr~~
  - careyq.cool.csr

因为使用的是 Nginx，所以就只用得到 Nginx 目录下的两个文件，分别是 `.crt` 文件 和 `.key` 文件。

## 上传证书到服务器

在服务器上创建一个文件夹，用于存放证书文件，我的路径为：`/home/SSL`。

使用 `git bash` 将两个文件上传至服务器的 `/home/SSL` 中。

```shell
scp 本地
文件路径 USERNAME@SERVER:/home/SSL
```

`USERNAME` 表示服务器用户。`SERVER` 表示服务器 IP。

## 证书安装

编辑 Nginx 的配置文件，我的为 `/etc/nginx/nginx.conf`，可通过 `nginx -t` 命令查看。在 `listen 80` 的 `server` 后面在添加如下：

```nginx
server {
    listen 443;
    server_name careyq.cool;     # 填写绑定证书的域名
    ssl on;
    ssl_certificate /home/SSL/1_bundle.crt;     # .crt 文件路径
    ssl_certificate_key /home/SSL/2_key.key;    # .key 文件路径
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    # 按照这个协议配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;     # 按照这个套件配置
    ssl_prefer_server_ciphers on;
    location / {
        root /home/hexo;    # 站点目录
        index  index.html index.htm;
    }
}
```

配置完成后，使用 `nginx -t` 命令检测是否有误，正确无误后，使用 `systemctl restart nginx.service` 重启 Nignx。再使用带 `https` 的域名进行访问即可。

| 配置文件参数        | 说明                                |
| ------------------- | ----------------------------------- |
| listen 443          | SSL 的访问端口号为 443              |
| ssl on              | 启用 SSL 功能                       |
| ssl_certificate     | 证书文件                            |
| ssl_certificate_key | 私钥文件                            |
| ssl_protocols       | 使用的协议                          |
| ssl_ciphers         | 配置加密套件，写法遵循 openssl 标准 |

## 使用全站加密

对于用户不知道网站可以进行 `https` 访问的情况下，可以让服务器自动把 `http` 的请求重定向到 `https`。可以通过 `js` 实现，也可以再服务器中实现跳转。Nginx 支持 `rewrite` 的，编辑 `Nginx` 的配置文件，在 `listen 80` 的 `server` 中添加语句，如下：

```bash
listen       80 default_server;
listen       [::]:80 default_server;
server_name  careyq.cool;
root         /home/hexo;

rewrite ^(.*) https://careyq.cool$1 permanent;   # 添加的语句
```

重启 Nginx，这样就可以实现 `80` 进来的请求重定向为 `https` 了。

---
title: Hexo 博客部署到腾讯云全流程
desc: ，看了许多文章，每篇文章总会有几个坑，由此总结一篇，把过程中踩到的坑记录下来，方便以后参阅。
categories: 博客搭建
date: 2018-09-28
url: blog-001
toc: true
---

原来我的博客是部署到「Github」和「Coding」上的，但是访问速度较慢，并且图床「七牛云」自 2018 年 7 月起，测试域名的生命周期只有 30 个自然日，这让维护博客造成了很大的麻烦，于是便准备使用服务器来进行部署。

## 服务器配置

> 因为 yum 源仓库的 Git 版本更新不及时，所以采用源码包进行安装。

### 安装依赖包

```bash
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum install  gcc perl-ExtUtils-MakeMaker
```

通过命令 `git --version` 可以看到，Git 当前的版本号为 `1.8.3.1`，太过陈旧，所以先把它移除了。

```bash
yum remove git
```

### 下载并解压

```bash
cd /usr/local/src   // 选择文件保存位置
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.19.0.tar.gz   // 下载链接
tar -zxvf git-2.19.0.tar.gz   // 解压
```

`git-2.19.0.tar.gz` 是目前最新版本，其他版本以及之后版本可 [到此](https://mirrors.edge.kernel.org/pub/software/scm/git) 进行查看。解压后，会在当前目录下自动生成一个名为 `git-2.19.0` 的文件夹，里面就是解压出来的文件。可通过命令 `ls` 进行查看。

### 编译安装

```
cd git-2.19.0   // 进入文件夹
make prefix=/usr/local/git all  // 编译源码
make prefix=/usr/local/git install  // 安装至 /usr/local/git 路径
```

编译时，由机器配置决定速度，请耐心等待。编辑环境变量配置文件

```bash
vim /etc/profile
```

在文件底部添加以下配置。

```bash
PATH=$PATH:/usr/local/git/bin   // git 的目录
export PATH
```

两个语句都需要加上，之后刷新环境变量

```bash
source /etc/profile
```

最后再使用 `git --version` 查看版本号，已经为 `2.19.0`。

### 创建 git 用户

```bash
adduser git
passwd git
chmod 740 /etc/sudoers
vim /etc/sudoers
```

找到如下内容：

```bash
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
```

在它下面添加一行，如下：

```bash
git ALL=(ALL) ALL
```

保存并退出，将权限修改回来。

```bash
chmod 400 /etc/sudoers
```

### 密钥配置

本地中，使用 `Git Bash` 创建密钥。

```bash
# 本地 windows gitBash
ssh-keygen -t rsa
```

一路回车，直至如下图所示

![](https://cdn.jsdelivr.net/gh/CareyQ/careyqx@master/article/184lrctzr32.png)

切换至 `git` 用户，创建 `.ssh` 文件夹以及 `authorized_keys` 文件，并将本地的 `id_rsa.pub` 文件内容粘贴到里面。

```bash
su git
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
```

修改权限

```bash
cd ~
chmod 600 .ssh/authorized_keys
chmod 700 .ssh
```

### 测试

在本地 Windows 上，使用 `Git Bash` 测试是否能连接上服务器。

```bash
ssh -v git@SERVER
```

其中 SERVER 为服务器 ip 地址。若出现以下错误提示，检查本地密钥位置是否存在 `known_hosts` 文件，将其删除重新测试。测试结果为不需要密码直接进入。

![](https://cdn.jsdelivr.net/gh/CareyQ/careyqx@master/article/wccpbtrzpmo.webp)

### 创建网站目录

创建一个目录用于作为网站的根目录

```bash
su root
mkdir /home/hexo    # 此目录为网站的根目录
```

赋予权限

```bash
chown git:git -R /home/hexo
```

### 安装配置 Nginx

> 需要 root 权限

```bash
yum install -y nginx    // 安装
systemctl start nginx.service     // 启动服务
```

使用 yum 安装的 nginx 在新版的 CentOS 中，需要使用`systemctl`，而不是直接使用 `service start nginx`。此时通过服务器的公网 IP 地址访问，可以看到 nginx 的欢迎页面，表示安装成功，如下图:

![](https://cdn.jsdelivr.net/gh/CareyQ/careyqx@master/article/1ch6wvqn4fkw.png)

使用 `nginx -t` 命令查看位置，一般为 `/etc/nginx/nginx.conf`
使用 `vim /etc/nginx/nginx.conf` 命令进行编辑，修改配置文件如下：

```nginx
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  staunchkai.com;    # 修改为自己的域名
    root         /home/hexo;    # 修改为网站的根目录

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

`root` 处的网站目录，需要自己创建，也是部署上传的位置。

注意使用 `nginx -t` 命令检查配置文件的语法是否出错。然后使用 `systemctl restart nginx.service` 命令重启服务即可。

### 自动化部署

创建一个裸仓库，裸仓库就是只保存 `git` 信息的 `Repository`, 首先切换到 `git` 用户确保 git 用户拥有仓库所有权，一定要加 `--bare`，这样才是一个裸库。

```bash
su root
cd /home/git   # 在 git 用户目录下创建
git init --bare blog.git
```

这时，`git` 用户的 `~` 目录下就存在一个 `blog.git` 文件夹，可使用 `ls` 命令查看。再修改 `blog.git` 的权限。

```bash
chown git:git -R blog.git
```

**使用 git-hooks 同步网站根目录**

在这使用的是 `post-receive` 这个钩子，当 `git` 有收发的时候就会调用这个钩子。 在 `blog.git` 裸库的 hooks 文件夹中，新建 `post-receive` 文件。

```bash
vim blog.git/hooks/post-receive
```

填入以下内容，其中 `/home/hexo` 为网站目录，根据自己的填入,保存退出。

```bash
#!/bin/sh
git --work-tree=/home/hexo --git-dir=/home/git/blog.git checkout -f
```

保存后，要赋予这个文件可执行权限。

```bash
chmod +x /home/git/blog.git/hooks/post-receive
```

## 本地配置

在本地，部署到服务器和部署到 `pages` 服务一样，需要先 `hexo g` 命令生成静态文件，通过 `hexo s` 命令能够正常进行本地访问，并且确保已经安装了 `hexo-deployer-git`。

### 配置 `_config.yml`

hexo 根目录下的 `_config.yml` 文件，找到 `deploy`。

```yml
deploy:
  type: git
  repo: git@SERVER:/home/git/blog.git # repository url
  branch: master # 分支
```

之后按照正常的流程进行部署即可。

```bash
hexo clean
hexo g
hexo d
```

## 常见错误

**git-receive-pack: 未找到命令**

```bash
hexo d
bash: git-receive-pack: command not found
fatal: Could not read from remote repository.
```

解决办法：

```bash
sudo ln -s /usr/local/git/bin/git-receive-pack  /usr/bin/git-receive-pack
```

**git-upload-pack: 未找到命令**

```bash
正克隆到 'blog'...
bash: git-upload-pack: command not found
fatal: Could not read from remote repository.
```

解决办法：

```bash
sudo ln -s  /usr/local/git/bin/git-upload-pack  /usr/bin/git-upload-pack
```

**不是一个 git 仓库**

```bash
fatal: 'XXXX/blog.git' does not appear to be a git repository
fatal: Could not read from remote repository.
```

解决办法：
确保 `_config.yml` 文件中 `deploy` 处的 `repo` 路径填写正确。

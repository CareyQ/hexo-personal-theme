---
title: 重装系统后，恢复 MySQL
categories: 数据库札记
desc: ，整个 MySQL 文件都还在，仅仅服务丢失了，用此方法就能恢复。
date: 2021-05-27
url: sql-003
---

MySQL 存放在磁盘中，重装系统后恢复数据库

1. 环境变量 `path` 中添加 MySQL 安装目录下的 `bin` 目录

2. 以管理员身份打开命令提示符，运行命令如下：

   ```shell
   $ mysqld -install
   Service successfully installed.

   $ net start mysql
   MySQL 服务正在启动 .
   MySQL 服务已经启动成功。
   ```

3. 这时候系统服务中就存在了 MySQL，正常使用即可。

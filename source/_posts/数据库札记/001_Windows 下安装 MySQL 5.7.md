---
title: Windows 下安装 MySQL 5.7
categories: 数据库札记
date: 2021-05-03
url: sql-001
---

本文描述的是在 Windows 下安装 MySQL 5.7.33 压缩版。[MySQL 下载地址](https://downloads.mysql.com/archives/community/)，该页面中可选择指定版本进行下载。

1. 将下载好的压缩包解压至自己要存放的位置，例如：`D:\Environment\mysql`

2. 在环境变量 `Path` 中添加 MySQL 目录下的 `bin` 文件夹，例如：`D:\Environment\mysql\bin`

3. 在根目录下添加配置文件 `my.ini`，完整路径应该为： `D:\Environment\mysql\my.ini`

4. 配置文件中添加如下信息

   ```ini
   [mysqld]
   basedir=D:/Environment/mysql/
   datadir=D:/Environment/mysql/data/
   port=3306
   skip-grant-tables
   ```

   - **basedir：**指的是 MySQL 的目录
   - **datadir：**指的是 MySQL 数据库存放地址，这个文件夹 **无需手动创建**
   - **skip-grant-tables：**用于首次跳过密码

5. 启动 **管理员模式 CMD**，输入 `mysqld --install` 命令进行安装

   ```shell
   $ mysqld --install
   Service successfully installed.
   ```

6. 输入 `mysqld --initialize-insecure --user=mysql` 命令初始化数据

   ```shell
   $ mysqld --initialize-insecure --user=mysql
   ```

7. 上一步完成后，命令行中没有任何提示信息，但是会生成配置文件中 **datadir** 配置的目录，例如：`D:\Environment\mysql\data`，有了该文件夹之后，使用命令 `net start mysql` 启动 MySQL

8. 输入 `mysql -u root -p` 进入 MySQL，提示输入密码，直接回车空密码即可

   ```shell
   $ mysql -u root -p
   Enter password:
   ```

   成功进入后，命令行提示应该如下：

   ```shell
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 3
   Server version: 5.7.33 MySQL Community Server (GPL)

   Copyright (c) 2000, 2021, Oracle and/or its affiliates.

   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   mysql>
   ```

9. 输入 SQL 语句，修改 MySQL 密码

   ```shell
   mysql> update mysql.user set authentication_string=password('123456') where user='root' and Host='localhost';
   Query OK, 1 row affected, 1 warning (0.00 sec)
   Rows matched: 1  Changed: 1  Warnings: 1
   ```

   - **123456** 改为自己想要的密码

10. 刷新权限

    ```shell
    mysql> flush privileges;
    Query OK, 0 rows affected (0.00 sec)
    ```

11. 删除配置文件中最后一句 `skip-grant-tables`，并使用命令重启 MySQL，在 MySQL 中需要先退出

    ```shell
    mysql> exit
    Bye

    $ net stop mysql
    MySQL 服务正在停止.
    MySQL 服务已成功停止。

    $ net start mysql
    MySQL 服务正在启动 .
    MySQL 服务已经启动成功。
    ```

12. 再次使用 `mysql -u root -p` 命令，输入密码即可进入，密码错误则提示密码不正确，到此，MySQL 安装完毕。

> 进入 MySQL 提示 `Can't connect to MySQL server on 'localhost' (10061)`，是因为 MySQL 服务未启动，使用 `net start mysql` 启动。

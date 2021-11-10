---
title: 通过 MySQL 数据库文件恢复数据库
categories: 数据库札记
desc: ，重装系统忘了导出 SQL 备份，只有磁盘上的数据库文件，可以通过此方法进行恢复。
date: 2021-05-03
url: sql-002
---

重装系统，没有导出 MySQL 执行文件，现通过 MySQL 数据库文件恢复数据库，原来的数据库版本为 `MySQL 5.7.32`，重装系统后的数据库版本为 `MySQL 5.7.33`。

1. 使用命令停止 MySQL 服务

   ```shell
   $ net stop mysql
   MySQL 服务正在停止.
   MySQL 服务已成功停止。
   ```

2. 拷贝旧数据库文件至新数据库，拷贝并覆盖 5 个配置文件，拷贝 N 个数据库文件
   - 数据库文件，例如之前的数据库名为 `myblog`，想要恢复它，就需要拷贝对应的文件夹至新数据库中
   - auto.cnf
   - ib_buffer_pool
   - ib_logfile0
   - ib_logfile1
   - ibdata1

> 建议先备份新数据库的 5 个配置文件

3. 使用命令启动 MySQL 服务

   ```shell
   $ net start mysql
   MySQL 服务正在启动 .
   MySQL 服务已经启动成功。
   ```

4. 再次进入 MySQL，数据已恢复

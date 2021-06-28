---
title: mysql命令
categories:
  - 数据库
tags:
  - mysql命令
translate_title: mysql-command
date: 2020-02-23 20:57:04
---

Windows服务

```mysql
-- 启动服务
net start mysql
-- 创建Windows服务(注意等号与值之间有空格)
sc create mysql binPath= mysql_bin_path
```

连接与断开服务器

```mysql
mysql -h 地址 -P 端口 -u 用户名 -p 密码

SHOW PROCESSLIST -- 显示那些线程正在运行
SHOW VARIABLES -- 显示系统变量信息
```

数据库操作

```mysql
-- 查看当前数据库
SELECT DATABASE();
-- 显示当前时间、用户名、数据库版本
SELECT now(),user(),version();
-- 创建库
CREATE DATABASE[IF NOT EXISTS] 数据库名 数据库选项
```


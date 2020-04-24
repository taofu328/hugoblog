---
title: "debian下新装Mysql登录提示MySQL Error: : 'Access denied for user 'root'@'localhost'的解决办法"
date: 2020-04-24T13:49:10+08:00
draft: false
categories: ["折腾",]
tags: [
    "debian",
	"linux",
    "mysql",
]
---

- 修改配置文件`sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf` ,在 `[mysqld` 中加入 `skip-grant-tables`  
- 重启mysql服务 `sudo systemctl restart mysql` ，使用空密码进入数据库,执行以下操作
```shell
mysql -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21-1 (Debian)

mysql>  use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update mysql.user set authentication_string=password('newpass') where user='root' and Host ='localhost';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> update user set plugin="mysql_native_password";
Query OK, 1 row affected (0.00 sec)
Rows matched: 4  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```

- 将配置文件中的 `skip-grant-tables` 删除，重启mysql服务,用新的密码登录
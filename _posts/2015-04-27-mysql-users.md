---
title: MySQL 用户管理
layout: post
comments: true
language: chinese
category: [mysql,database]
keywords: mysql,database,数据库,权限管理
description: MySQL 权限管理系统的主要功能是证实连接到一台给定主机的用户，并且赋予该用户在数据库上的相关权限，在认证的时候是通过用户名+主机名定义。 此，简单介绍下 MySQL 的权限管理，以及设置相应的用户管理。
---

MySQL 权限管理系统的主要功能是证实连接到一台给定主机的用户，并且赋予该用户在数据库上的相关权限，在认证的时候是通过用户名+主机名定义。

在此，简单介绍下 MySQL 的权限管理，以及设置相应的用户管理。

<!-- more -->

![mysql]({{ site.url }}/images/databases/mysql/account-logo.png "mysql"){: .pull-center width="75%" }

## 权限管理

简单介绍一些用户管理的操作。

### 常用操作

在设置权限时可以使用通配符，其中 % 表示可以匹配任意长度字符， _ 表示可以匹配单字符，常见的操作列举如下：

{% highlight text %}
----- 查看帮助
mysql> help account management;
mysql> help create user;

----- 常用操作
mysql> SELECT current_user();                                        ← 查看当前用户
mysql> SELECT user,host,password FROM mysql.user;                    ← 查看用户信息
mysql> SHOW GRANTS;                                                  ← 当前用户权限，会生成SQL语句
mysql> CREATE USER 'user'@'host' IDENTIFIED BY 'password';           ← 创建用户
mysql> DROP USER 'user'@'host';                                      ← 删除用户
mysql> RENAME USER 'user'@'host' TO 'fool'@'host';                   ← 修改用户名

----- 修改密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new-password';   ← 修改密码(recommand)
mysql> SET PASSWORD FOR 'root'@'localhost'=PASSWORD('new-password'); ← 修改密码
mysql> UPDATE mysql.user SET password=PASSWORD('new-password')
       WHERE USER='root' AND Host='127.0.0.1';
mysql> UPDATE mysql.user SET password='' WHERE user='root';          ← 清除密码
mysql> FLUSH PRIVILEGES;
$ mysqladmin -uROOT -pOLD_PASSWD password NEW_PASSWD                 ← 通过mysqladmin修改
$ mysqladmin -uROOT -p flush-privileges

----- 权限管理
mysql> GRANT ALL PRIVILIGES ON *.* TO 'user'@'%' [IDENTIFIED BY 'password'];
mysql> GRANT ALL PRIVILIGES ON [TABLE | DATABASE] student,course TO user1,user2;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY, ALTER,
       DROP, REFERENCES, INDEX, CREATE VIEW, SHOW VIEW, CREATE ROUTINE,
       ALTER ROUTINE, EXECUTE
       ON db.tbl TO 'user'@'host' [IDENTIFIED BY 'password'];
mysql> GRANT ALL ON sampdb.* TO PUBLIC WITH GRANT OPTION;            ← 所有人，可以授权给其他人
mysql> GRANT UPDATE(col),SELECT ON TABLE tbl TO user;                ← 针对列赋值
mysql> SHOW GRANTS [FOR 'user'@'host'];                              ← 查看权限信息
mysql> REVOKE ALL ON *.* FROM 'user'@'host';                         ← 撤销权限
mysql> REVOKE SELECT(user, host), UPDATE(host) ON db.tbl FROM 'user'@'%';

----- 查看用户或者密码为空的用户
mysql> SELECT user, host, password FROM mysql.user
       WHERE (user IS NULL OR user='') OR (password IS NULL OR password='');

----- 重命名用户
mysql> RENAME USER 'user1'@'%' TO 'user2'@'%';
{% endhighlight %}

WITH GRANT OPTION 表示可以将被授予的权力再次授权给其他人；实际上，在上述操作中，如下的步骤是相同的。

{% highlight text %}
mysql> CREATE USER 'user' IDENTIFIED BY 'password';
mysql> GRANT privileges TO 'user'@'host' [WITH GRANT OPTION];
mysql> FLUSH PRIVILEGS;

mysql> GRANT privileges  TO 'user'@'host' IDENTIFIED BY 'password' [WITH GRANT OPTION];
mysql> FLUSH PRIVILEGS;
{% endhighlight %}

也就是将创建用户和授权是否分开处理。

### 用户设置

![mysql]({{ site.url }}/images/databases/mysql/account-logo2.png "mysql"){: .pull-right width="18%" }

为了最小化用户权限，通常可以设置四类：

* monitor 对内部表的只读权限，用于监控系统采集监控数据；
* readonly 业务表的只读权限，比如用于运营、开发排查问题等；
* admin 基本是最高权限，一些运维人员使用的帐号，例如建表、帐号管理等操作；
* foobar 业务帐号，也就是应用使用的帐号，业务业务表有增删改查的权限。

接下来，我们看看如何管理这些帐号。

#### monitor

创建完用户之后，默认是有 information_schema 库的读权限的。

{% highlight text %}
mysql> CREATE USER 'monitor'@'%' IDENTIFIED BY 'password';
mysql> SHOW GRANTS FOR 'monitor'@'%';
mysql> GRANT SELECT ON performance_schema.* TO 'monitor'@'%';
mysql> GRANT SELECT ON mysql.* TO 'monitor'@'%';

mysql> REVOKE SELECT ON mysql.* FROM 'monitor'@'%';
mysql> REVOKE SELECT ON performance_schema.* FROM 'monitor'@'%';
mysql> DROP USER 'monitor'@'%';
{% endhighlight %}

#### readonly

与 monitor 用户相同，不过是针对的业务数据库授权。

#### admin

只针对管理平面的机器设置权限，也就是指定 IP 地址。

{% highlight text %}
mysql> CREATE USER 'admin'@'IP' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'IP';

mysql> REVOKE ALL PRIVILEGES ON *.* FROM 'admin'@'IP';
mysql> DROP USER 'admin'@'IP';
{% endhighlight %}

#### root

保留只允许本地登陆的 root 用户，Always have a plan B 。

{% highlight text %}
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
{% endhighlight %}

Plan C 是免密码登陆。

## 其它

在此，主要列举一些常见的用户相关操作。

### 重置 root 用户秘密

如果忘记了 root 用户，可以通过如下方法进行更改。

{% highlight text %}
----- 1. 停止mysql服务器
# systemctl stop mysqld
# /opt/mysql-5.7/bin/mysqladmin -uroot -p'init-password' shutdown
Shutting down MySQL..     done

----- 2. 获取跳过认证的启动参数
# mysqld --help --verbose | grep 'skip-grant-tables' -A1
    --skip-grant-tables Start without grant tables. This gives all users FULL
                          ACCESS to all tables.

----- 3. 启动服务器，跳过认证
# mysqld --skip-grant-tables --user=mysql &
[1] 10209

----- 4. 取消密码
mysql> UPDATE mysql.user SET password='' WHERE user='root';
Query OK, 2 rows affected (0.00 sec)
Rows matched: 2  Changed: 2  Warnings: 0
{% endhighlight %}

然后正常重启即可，空密码登陆，再设置好密码。

## 参考

官方文档 [MySQL Reference Manual - Security](http://dev.mysql.com/doc/refman/en/security.html)，主要是与安全相关的内容，包括了用户管理。

{% highlight text %}
{% endhighlight %}

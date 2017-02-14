---
title: MySQL 杂项
layout: post
comments: true
language: chinese
category: [mysql,database]
keywords: mysql,杂项
description: 简单记录下 MySQL 常见的一些操作。
---

简单记录下 MySQL 常见的一些操作。

<!-- more -->

## 初始化密码

MySQL 5.7 在初始化时会生成默认的密码，当然可以在初始化的时候使用 ```--initialize-insecure``` 参数不设置密码，不过通过 mysql 客户端登陆的时候，仍会提示输入密码。

对于自动化脚本，最好的方式是在启动 MySQL 服务器时使用 ```--skip-grant-tables``` 参数，然后更新密码。

{% highlight text %}
----- MySQL5.7.6及以上版本
mysql> UPDATE mysql.user SET authentication_string=PASSWORD('newpass') WHERE user='root';
----- MySQL 5.7.5及之前版本
mysql> UPDATE mysql.user SET password=PASSWORD('newpass') WHERE user='root';

----- 授权远程访问
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'newpass' WITH GRANT OPTION;
{% endhighlight %}

注意，如果使用了 ```--skip-grant-tables``` 参数，如果使用如下密码更新时，将会报错。

{% highlight text %}
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new-password';
{% endhighlight %}

<!--
set global validate_password_policy=0; set global validate_password_mixed_case_count=2;
-->

{% highlight text %}
{% endhighlight %}

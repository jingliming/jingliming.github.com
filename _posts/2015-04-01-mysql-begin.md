---
title: MySQL 写在开头
layout: post
comments: true
language: chinese
category: [mysql,database]
keywords: mysql
description: 保存一下经常使用的经典 MySQL 资源。
---

保存一下经常使用的经典 MySQL 资源。

<!-- more -->

## 常用网站

记录一些常用的网站。

### 官方网站

* [MySQL Documentation](http://dev.mysql.com/doc/)，MySQL 官方网站的参考文档，经常使用的包括了 [MySQL Reference Manual](http://dev.mysql.com/doc/refman/en/)、[MySQL Internals Manual](http://dev.mysql.com/doc/internals/en/)，还包括了一些 API 接口。

* [MySQL-InnoDB](https://blogs.oracle.com/mysqlinnodb/) InnoDB 的官方 blog ，很多不错的文章。

* [MySQL Server Blog](http://mysqlserverteam.com/)、[MySQL High Availability](http://mysqlhighavailability.com/) 。

* [Contributing Code to MySQL](https://community.oracle.com/docs/DOC-914911)，介绍如何向 MySQL 社区贡献代码。

### 数据库排名

关于当前数据库的排名比较有权威的是 [DB-Engines](http://db-engines.com/)，包括了关系型数据库、时序型数据库等等，其排名的数据计算是依据几个不同的因素，其中包括了：

* Google 以及 Bing 搜索引擎的关键字搜索数量；
* Google Trends 的搜索数量；
* Indeed 网站中的职位搜索量；
* LinkedIn 中提到关键字的个人资料数；
* Stackoverflow 上相关的问题和关注者数量。

其中各种类型数据库的综合排名可以参考 [DB-Engines Ranking](http://db-engines.com/en/ranking)，各个数据库的排名趋势则可以参考 [DB-Engines Ranking Trend](http://db-engines.com/en/ranking_trend) 。


## Blogs

一些比较常见、经典的 Blogs 。

### Jeremy Cole

目前在 Google 任职的大牛，分别在 MySQL AB、Yahoo、Twitter、Google 待过，看他的 blog 竟然还玩过 AVR 的 ^_^

可以参考其个人 blog：[Jeremy Cole](https://blog.jcole.us/) ，很大一部分是与 InnoDB 文件格式相关的内容，写的不错。

其维护了一个使用 Ruby 语言编写的 innodb_ruby 工具，用来查看 Innodb 的数据文件状态，当然只适用于实验测试环境。

innodb_ruby 工具的源码可以参考 [Github](https://github.com/jeremycole/innodb_ruby/)，其安装、使用方法可以参考 [Github WiKi](https://github.com/jeremycole/innodb_ruby/wiki) 。


## 经典文章

* [How does a relational database work](http://coding-geek.com/how-databases-work/)，一篇很好的文章，对关系型数据库的初步介绍。

* [An Outline for a Book on InnoDB](http://www.xaprb.com/blog/2015/08/08/innodb-book-outline/)，关于 InnoDB 的一些关键技术点。


## 经典课程

* [Stanford 数据库课程](http://web.stanford.edu/class/cs245/)，介绍了一些常用的数据库工具，如 [Innodb diagrams](https://github.com/jeremycole/innodb_diagrams)、[Innodb Ruby](https://github.com/jeremycole/innodb_ruby/) 。

<!--
<a href="http://www.innomysql.net/">Inside MySQL</a> 网易姜承尧的 blog 。<br><br>
<a href="https://github.com/percona/tokudb-engine">tokudb-engine github</a>，据说一个很牛掰的存储引擎，与 InnoDB 类似，percona 实现的。<br><br>
<a href="http://www.gpfeng.com/">Learn AND live</a> <br><br>
<a href="http://mysql.taobao.org/index.php?title=%E9%A6%96%E9%A1%B5">淘宝MySQL</a>，官方 blog 。<br><br>
http://cn.planet.mysql.com/
hotpu-meeting.b0.upaiyun.com/2014dtcc/post_pdf/hedengcheng.pdf




http://www.ywnds.com/?cat=31
http://mysqllover.com/?p=594
http://siddontang.com/?p=594
http://keithlan.github.io/      binlog
http://hatemysql.com/           binlog
-->


{% highlight text %}
{% endhighlight %}

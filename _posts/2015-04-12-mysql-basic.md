---
Date: October 19, 2013
title: MySQL 基本介绍
layout: post
comments: true
language: chinese
category: [mysql,database]
---

在此主要介绍下与数据库相关的一些操作，其中有一部分是与 MySQL 相关的功能。

<!-- more -->


## 简介

![sql logo]({{ site.url }}/images/databases/mysql/sql-logo.jpg "sql logo"){: .pull-right }

首先简单介绍一下 SQL (Structured Query Language) 相关的标准；SQL-92 是最常用的 SQL 标准，其定义了一些常见的语法等，详细的可参考 [SQL-92](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt) ，也可查看 [本地文档](/reference/mysql/SQL1992.txt) 。

其中 SQL 常见的操作包括如下：

* DDL：数据库模式定义语言，create 。
* DML：数据操纵语言，insert、delete、update 。
* DCL：数据库控制语言 ，grant、remove 。
* DQL：数据库查询语言，select 。

不过不同的数据库对 SQL-92 标准会有所扩展，这也就造成了一些数据库操作的不兼容，或者是一些细节上的差异。

对于 MySQL，在 Linux 主机上的默认存储位置为 /var/lib/mysql/ ； FreeBSD 在 /var/db/mysql；如果想要修改可以通过符号链接即可。

如果是单独进行安装的，可以通过如下命令查看 MySQL 是否安装成功。

{% highlight text %}
$ mysqladmin --version
{% endhighlight %}

### 示例数据库

通常最基本的是熟悉简单的 join 操作，以及基本的 CRUD(Create, Retrieve, Update, Delete) 操作，如下包括了很多教程示例。

下面的示例可以参考 [MySQLTutorial](http://www.mysqltutorial.org/)，数据库的数据也可以从 [本地下载](/reference/mysql/from_mysqltutorial.sql) ，然后通过 source 导入。下图是对应的 ER 图，其中关于 ER-Diagram 相关的介绍可以参考 [ER Diagram Symbols and Meaning (LucidChart)](https://www.lucidchart.com/pages/ER-diagram-symbols-and-meaning) 。

![sample sql]({{ site.url }}/images/databases/mysql/sample_database_store.png "sample sql"){: .pull-center width="60%" }

上述是一个销售汽车的模型，包括了常见的模型以及使用数据。

<!--
Customers，保存了用户的信息。
Products，产品信息，也就是车的信息。
ProductLines: stores a list of product line categories.
Orders: stores  sales orders placed by customers.
OrderDetails: stores sales order line items for each sales order.
Payments: stores payments made by customers based on their accounts.
Employees: stores all employee information as well as the organization structure such as who reports to whom.
Offices: stores sales office data.
-->


另外，可以参考 [employee](https://launchpad.net/test-db/) 的示例数据库，现在已经迁移到 [github](https://github.com/datacharmer/test_db)，其文档位于 [Employees Sample Database](http://dev.mysql.com/doc/employee/en/) ，可以直接从 [本地下载](/reference/mysql/test_db-master.zip) 。

![employees schema]({{ site.url }}/images/databases/mysql/employees-schema.png "employees schema"){: .pull-center width="50%"}

其中 README.md 中包括了如何进行安装。


<!---
http://blog.csdn.net/imzoer/article/details/8277797
-->

{% highlight text %}
{% endhighlight %}

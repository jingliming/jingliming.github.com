---
title: MySQL CAPI 接口
layout: post
comments: true
language: chinese
category: [mysql,database]
keywords: mysql,c,api
description: 在编写 MySQL 客户端程序时，最常见的就是如何连接以及关闭 MySQL，这里需要注意，如果使用不当将会造成内存泄漏。这里，简单介绍 MySQL 中如何通过 C 进行编程。
---

在编写 MySQL 客户端程序时，最常见的就是如何连接以及关闭 MySQL，这里需要注意，如果使用不当将会造成内存泄漏。

这里，简单介绍 MySQL 中如何通过 C 进行编程。

<!-- more -->

## 简介


单线程环境，一般使用 `-lmysqlclient` 链接 MySQL 客户库，其中 `mysql_init()` 会自动调用 `my_library_init()` 初始化 MySQL 库，所以不需要，需要注意的是这两个函数都是非线程安全的。

示例代码如下。

{% highlight c %}
void wrap_mysql_connect(MYSQL *mysql_conn)
{
	my_init();
	if (NULL == mysql_init(mysql_conn)) {
		write_log("Colud not init mysql.");
		return;
	}

	if (!mysql_real_connect(mysql_conn, g_mysql->host, g_mysql->username,
			g_mysql->passwd, g_mysql->dbname, g_mysql->port, NULL, 0)) {
		write_log("Mysql connect error : .", mysql_error(mysql_conn));
	}
}

void wrap_mysql_close(MYSQL *mysql_conn)
{
	mysql_close(mysql_conn);
	mysql_conn = NULL;
}
{% endhighlight %}

多线程环境，一般调用 `-lmysqlclient_r` 安全类库，需要在各个线程中调用 `mysql_library_init()` `mysql_library_end()` 来分配和释放 MySQL 资源，或者增加线程锁保护资源，否则会造成内存泄漏。

示例代码如下。

{% highlight c %}
void wrap_mysql_connect(MYSQL *mysql_conn)
{
	my_init();
	if (NULL == mysql_init(mysql_conn)) {
		write_log("Colud not init mysql.");
		return;
	}

	if (mysql_library_init(0, NULL, NULL)) {
		write_log("Could not initialize mysql library.");
		return;
	}

	if (!mysql_real_connect(mysql_conn, g_mysql->host, g_mysql->username,
			g_mysql->passwd, g_mysql->dbname, g_mysql->port, NULL, 0)) {
		write_log("Mysql connect error : .", mysql_error(mysql_conn));
	}
}


void pa_mysql_close(MYSQL *mysql_conn)
{
	mysql_close(mysql_conn);
	mysql_conn = NULL;
	mysql_library_end();
}
{% endhighlight %}


在使用 MySQL 多线程时，需要注意如下的问题：

1. `mysql_library_init()` 和 `mysql_library_end()` 必须要在同一个线程中，否则会出现 `Error in my_thread_global_end(): 1 threads didn't exit` 的报错。
2. 当通过 `mysql_ping()` 检测到链接断开后，直接调用 `mysql_real_connect()` 重新链接会导致本次链接的部分内存没有释放。
3. 如果设置成自动重新链接，那么在其它线程通过 `mysql_ping()` 重新链接时，同样会存在问题。


## 简单示例

{% highlight c %}
#include <mysql.h>
#include <my_global.h>

int main(int argc, char **argv)
{
        printf("MySQL client version: %s\n", mysql_get_client_info());

        MYSQL *conn = mysql_init(NULL);
        if (conn == NULL) {
                printf("Error %u: %s\n", mysql_errno(conn), mysql_error(conn));
                exit(1);
        }

        /* arguments: host user password database port unix_socket client_flag */
        if (mysql_real_connect(conn, "localhost", "root", "Huawei@123", NULL, 0, NULL, 0) == NULL) {
                printf("Error %u: %s\n", mysql_errno(conn), mysql_error(conn));
                mysql_close(conn);
                exit(1);
        }

        if (mysql_ping(conn)) {
                printf("Error: lost connection\n");
                return 1;
        }

        if (mysql_query(conn, "DROP DATABASE IF EXISTS foobardb")) {
                printf("Error %u: %s\n", mysql_errno(conn), mysql_error(conn));
                mysql_close(conn);
                exit(1);
        }

        if (mysql_query(conn, "SHOW DATABASES")) {
                printf("Error %u: %s\n", mysql_errno(conn), mysql_error(conn));
                mysql_close(conn);
                exit(1);
        }

        MYSQL_RES *result = mysql_store_result(conn);
        int num_fields = mysql_num_fields(result), i;
        MYSQL_ROW row;
        while ((row = mysql_fetch_row(result))) {
                for(i = 0; i < num_fields; i++)
                        printf("%s ", row[i] ? row[i] : "NULL");
                printf("\n");
        }
        mysql_free_result(result);

        mysql_close(conn);

        return 0;
}
{% endhighlight %}

然后通过如下方式进行编译。

{% highlight text %}
hello: main.c
	gcc -o hello main.c `mysql_config --cflags --libs`
{% endhighlight %}


<!--
mysql_real_escape_string() 和 mysql_real_string() 两者从入参实际上就可以做区分，前者需要链接 + 字符串，后者只需要字符串。前者会根据当前的符号集进行转换，也就是可以正常的处理一些多字符的字符集，而后者可能会在一个字符中间插入转义符。

不过除了采用 escaping ，建议使用参数化查询。

-->


<!--
MySQL timeout知多少
http://blog.csdn.net/sgbfblog/article/details/44262339
-->



{% highlight text %}
{% endhighlight %}

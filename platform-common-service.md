---
title: 基础服务
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---

服务端，支持 HTTP 请求。

### 文档服务器搭建

增加 HTTP/FTP 服务器，分别使用 nginx 和 vsftpd 搭建。

#### HTTP 服务

在 `/etc/nginx/conf.d` 目录下包含了各种配置文件，如果要增加新配置，可以直接在该目录下新增配置文件 `fileserver.conf` 。

{% highlight text %}
server {
	listen 80;                       # 监听端口
	server_name 127.0.0.1;           # 如果没有DNS解析，可以设置IP地址
	client_max_body_size 4G;         # 设置最大文件大小
	charset utf-8;                   # 防止出现中文乱码
	root /files;                     # 指定相对路径的根目录，如下的location会相对该路径
	location /packages {             # 实际存放文件的目录为/files/packages/
		auth_basic "Restricted"; # 输入密码时的提示语
		auth_basic_user_file /etc/nginx/pass_file; # 认证时用户密码文件存放路径
		autoindex on;            # 自动生成文件索引
		autoindex_exact_size on; # 显示文件大小
		autoindex_localtime on;  # 显示本地文件时间
	}
}
{% endhighlight %}

然后通过 `nginx -t` 检查语法是否正确，通过 `nginx -s reload` 重新加载配置；然后通过 [http://127.0.0.1/files/packages](http://127.0.0.1/files/packages) 访问。

另外，简单的可以采用 python 提供模块，不过只能采用单线程。

{% highlight text %}
----- 启动一个简单的文件服务器
$ python -m SimpleHTTPServer 9000

$ curl http://127.0.0.1:9000
{% endhighlight %}

可以直接通过 [http://127.0.0.1:8000](http://127.0.0.1:9000) 地址访问，可以看到目录下的文件列表。


#### FTP 服务

直接启动 vsftpd 服务器即可。

### 后台服务搭建

这里基于 Flask 和 MySQL 搭建后台服务器。

#### MySQL

{% highlight text %}
----- 安装社区版本MySQL，新增配置
# rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
# cat << EOF > /etc/my.cnf
[client]
password        = new_password
port            = 5506
socket          = /tmp/mysql-5506.sock
[mysqld]
user            = mysql
port            = 5506
socket          = /tmp/mysql-5506.sock
basedir         = /usr
datadir         = /tmp/data-5506
pid-file        = /tmp/data-5506/mysqld.pid
EOF

----- 初始化数据，可通过空白密码登陆
# mkdir /tmp/data-5506
# /usr/sbin/mysqld --initialize-insecure --basedir=/usr --user=mysql \
    --datadir=/tmp/data-5506
# systemctl start mysqld

----- 登陆并修改密码，第一次登陆时密码空白即可，分别对应了5.6以及5.7版本
$ mysql -h 127.1 -p
mysql> UPDATE mysql.user SET password=PASSWORD('new_password') WHERE user='root';
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
mysql> FLUSH PRIVILEGES;
{% endhighlight %}





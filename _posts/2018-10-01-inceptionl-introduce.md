---
title: inceptionk安装和使用
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: linux,MySQL,Inception
description: MySQL Inception是数据库管理员的工具。它允许DBA构建好的SQL语句，在只读数据集上测试它们，并最终针对生产数据库运行这些SQL语句，并且能够在SQL语句出于某种原因未达到预期结果时进行回滚。
---

MySQL Inception是数据库管理员的工具。它允许DBA构建好的SQL语句，在只读数据集上测试它们，并最终针对生产数据库运行这些SQL语句，并且能够在SQL语句出于某种原因未达到预期结果时进行回滚。

<!-- more -->

## Inception介绍
{% highlight text %}
MySQL Inception是数据库管理员的工具。它允许DBA构建好的SQL语句，在只读数据集上测试它们，
并最终针对生产数据库运行这些SQL语句，并且能够在SQL语句出于某种原因未达到预期结果时进行回滚。
{% endhighlight %}
## Inception下载

{% highlight text %}
 [github地址](https://github.com/mysql-inception/inception)
获取 git clone https://github.com/mysql-inception/inception.git
{% endhighlight %}

## 依赖环境安装
### bison

`bison最好使用2.6以前的版本，不然会出现inception版本安装失败`

{% highlight text %}
----获取
wget http://ftp.gnu.org/gnu/bison/bison-2.5.tar.gz
----安装
./configure --prefix=/usr --docdir=/usr/local/bin/bison
meke
make check
make install
----查看当前系统是否自带2.6以上的bison版本
which bison
/opt/compiler/gcc-4.8.2/bin/bison
----切换默认的bison
mv /opt/compiler/gcc-4.8.2/bin/bison /opt/compiler/gcc-4.8.2/bin/bison.3.0
mv /usr/local/bin/bison /opt/compiler/gcc-4.8.2/bin/bison
{% endhighlight %}

### cmake
{% highlight text %}
----获取
wget https://cmake.org/files/v3.11/cmake-3.11.1-Linux-x86_64.tar.gz
----安装
tar -zxvf cmake-3.11.1-Linux-x86_64.tar.gz
cd cmake-3.11.1
./bootstrap
gmake
gmake install
cmake --version
{% endhighlight %}

### curses5-dev

{% highlight text %}
yum install ncurses-devel.x86_64
{% endhighlight %}

## Inception安装
{% highlight text %}
sh inception_build.sh debug 
{% endhighlight %}

## Inception验证
### 启动
{% highlight text %}
vim inc.cnf
[inception]
general_log=1
general_log_file=inception.log
port=6669
socket=/data/workspace/inception_data/inc.socket
character-set-client-handshake=0
character-set-server=utf8
inception_remote_system_password=root
inception_remote_system_user=wzf1
inception_remote_backup_port=3306
inception_remote_backup_host=127.0.0.1
inception_support_charset=utf8mb4
inception_enable_nullable=0
inception_check_primary_key=1
inception_check_column_comment=1
inception_check_table_comment=1
inception_osc_min_table_size=1
inception_osc_bin_dir=/data/temp
inception_osc_chunk_time=0.1
inception_enable_blob_type=1
inception_check_column_default_value=1

debug/mysql/bin/Inception --defaults-file=inc.cnf
debug为安装的目录
{% endhighlight %}

### 连接
{% highlight text %}
mysql -uroot -h127.0.0.1 -P6669
inception get variables;
{% endhighlight %}

### 问题
装完之后，yum出现问题
{% highlight text %}
There was a problem importing one of the Python modules required to run yum. The error leading to this problem was:/usr/lib64/libssl.so.10: symbol private_ossl_minimum_dh_bits, version libcrypto.so.10 not defined in file libcrypto.so.10 with link time reference
Please install a package which provides this module, or verify that the module is installed correctly.
{% endhighlight %}
网上查询之后确定是openssl的硬连接出现问题，去目录`/usr/lib64`查看
{% highlight text %}
ll libcrypto.*
lrwxrwxrwx 1 root root      14 Aug 24  2017 libcrypto.so.10 -> libcrypto.so.4
-rwxr-xr-x 1 root root 1971488 Mar 23  2017 libcrypto.so.1.0.1e
-rwxr-xr-x 1 root root 1967392 Aug 23  2017 libcrypto.so.4
{% endhighlight %}

发现 libcrypto.so.10链接的so版本不对，改为1.0.1.e
{% highlight text %}
rm libcrypto.so.10
rm: remove symbolic link 'libcrypto.so.10'? yes
[root@gzhxy-bcc-init0000002774 lib64]# ll libcrypto.*
-rwxr-xr-x 1 root root 1971488 Mar 23  2017 libcrypto.so.1.0.1e
-rwxr-xr-x 1 root root 1967392 Aug 23  2017 libcrypto.so.4

ln -s libcrypto.so.1.0.1e libcrypto.so.10
[root@gzhxy-bcc-init0000002774 lib64]# ll libcrypto.*
lrwxrwxrwx 1 root root      19 Apr 26 11:25 libcrypto.so.10 -> libcrypto.so.1.0.1e
-rwxr-xr-x 1 root root 1971488 Mar 23  2017 libcrypto.so.1.0.1e
-rwxr-xr-x 1 root root 1967392 Aug 23  2017 libcrypto.so.4
{% endhighlight %}

重启sshd
{% highlight text %}
service sshd restart
Stopping sshd:                                             [  OK  ]
Starting sshd:                                             [  OK  ]
{% endhighlight %}

验证成功
{% highlight text %}
yum --version
3.2.29
  Installed: rpm-4.8.0-37.el6.x86_64 at 2014-09-01 09:55
  Built    : CentOS BuildSystem <http://bugs.centos.org> at 2013-11-22 11:38
  Committed: Panu Matilainen <pmatilai@redhat.com> at 2013-09-12

  Installed: yum-3.2.29-81.el6.centos.noarch at 2017-08-16 09:27
  Built    : CentOS BuildSystem <http://bugs.centos.org> at 2017-03-22 05:32
  Committed: Johnny Hughes <johnny@centos.org> at 2017-03-21

  Installed: yum-plugin-fastestmirror-1.1.30-14.el6.noarch at 2014-09-01 09:55
  Built    : CentOS BuildSystem <http://bugs.centos.org> at 2012-06-22 12:23
  Committed: Zdenek Pavlas <zpavlas@redhat.com> at 2012-04-26
{% endhighlight %}


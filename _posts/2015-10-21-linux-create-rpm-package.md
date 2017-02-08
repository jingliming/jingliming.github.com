---
title: RPM 包制作
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: linux,rpm,制作
description: RPM Package Manager 简称 RPM，这一文件格式在历史名称上虽然打着 RedHat 标志，但其原始设计理念是开放式的，现在包括 OpenLinux、SuSE、Turbo Linux 等多个 Linux 分发版本都有采用，可以算是公认的行业标准了。在此介绍下如何制作 RPM 包，尤其是如何写 .spec 配置文件，以及常见的技巧。
---

RPM Package Manager 简称 RPM，这一文件格式在历史名称上虽然打着 RedHat 标志，但其原始设计理念是开放式的，现在包括 OpenLinux、SuSE、Turbo Linux 等多个 Linux 分发版本都有采用，可以算是公认的行业标准了。

在此介绍下如何制作 RPM 包，尤其是如何写 .spec 配置文件，以及常见的技巧。

<!-- more -->

## 简介

若要构建一个标准的 RPM 包，需要创建 .spec 文件，其中包含软件打包的全部信息。然后，使用 rpmbuild 命令，按照 spec 文件的配置，系统会按照步骤生成最终的 RPM 包。

另外，需要注意的是，在使用时，需要使用普通用户，一定不要用 root 用户。

结下来，看看如何编译制作 RPM 包。

## 环境配置

接下来，看看如何配置一步步制作 RPM 包。

### 安装工具

首先，在 CentOS 中配置环境。

{% highlight text %}
----- 安装环境，需要rpm-build工具来打包，该包只依赖于gdb以及rpm版本
# yum install rpm-build -y

----- 新建所需目录
$ mkdir -pv ~/rpm-maker/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS}
{% endhighlight %}

### 配置环境

打包相关的配置保存在宏文件 (macrofiles) 中，制作包时的操作都在 topdir 指定的目录下，可以通过如下方式查看。

{% highlight text %}
--- 查看所有的配置文件顺序
$ rpmbuild --showrc | grep macros

--- 确认上述的根工作目录，两种方式相同
$ rpmbuild --showrc | grep ': _topdir'
$ rpm --eval '%_topdir'

--- 设置topdir变量，然后再次验证下，确认已经修改为目标目录
$ echo "%_topdir %{getenv:HOME}/rpm-maker" > ~/.rpmmacros
$ rpm --eval '%_topdir'
--- 也可以在制作rpm包时通过--define指定
$ rpmbuild --define '_topdir rpm-maker' -ba package.spec
{% endhighlight %}

下载 MySQL 的源码，并保存在 SPECS 目录下，整体目录结构如下。

{% highlight text %}
$ tree ~/rpm-maker
.
|-- BUILD                      ← 解压后的tar.gz包
|-- BUILDROOT                  ← 临时安装目录，会打包该目录下文件
|-- RPMS                       ← 编译后的RPM包
|-- SOURCES                    ← 所需的文件
|   `-- mysql-xxx.tar.gz       ← 包括了源码包
|-- SPECS                      ← *.spec编译脚本
|   |-- mysql-xxx.spec
|   `-- mysql-yyy.spec
`-- SRPMS                      ← src格式的rpm包存放的位置
{% endhighlight %}

### 执行命令

接下来，看看如何生成 rpm 包，也就是使用 rpmbuild 命令，过程可以分为多个阶段。

{% highlight text %}
$ rpmbuild --bb mysql.spec    

使用参数：
   -bp 执行到pre
   -bc 执行到build段
   -bi 执行install段

   --quiet 默认会输出每次执行的shell命令，此时忽略
   --test 只生成构建的脚本
{% endhighlight %}

其中，```--test``` 参数可以用来测试脚本是否正确，默认会保存在 /var/tmp 目录下，该路径可以通过 rpmrc 文件中的 tmppath 参数设置，或者直接设置 %_tmppath /tmp 。

<!--
-bb    制作成二进制                               // # sudo rpmbuild –bb mysql.spec
-bs    源码形式
-ba    源码和二进制两种形式

-bl    制作后检测buildroot生成但没有包含到rpm包中的文件，注意如果生成未包含进去会出错

-ba 既生成src.rpm又生成二进制rpm
-bs 只生成src的rpm
-bb 只生二进制的rpm

-bl 检测有文件没包含
-->

一般来说，执行的顺序为 rpmbuild -bp，再 -bc 再 -bi，如果没问题，rpmbuild -ba 生成 src 包与二进制包，使用时通过 ```rpm -ivh xxx.rpm;``` 以及 ```rpm -e xxx.rpm``` 进行安装卸载。

## SPEC 文件

接下来准备 spec 文件，也是核心的内容，该文件包括三部分：介绍部分，编译部分，files 部分。接下来，我们以 MySQL 为例，看看到底是如何制作 RPM 包的。

<!--
introduction section部分，包括一些变量定义：

###################################################
# File   : mysql-company-x.x.x-release.x86_64.spec
# Author : wyett
# Date   : Oct 15,2013
###################################################


#--- 定义变量，方便后面使用；其中_topdir指定工作根目录
%define _topdir /home/wyett/mysql
%define name mysql
%define version x.x.x
%define release dba.4
%define serial 1
%define userpath /usr/local/mysql
%define myusr mysql
%define mygrp mysql
%define buildroot %{_topdir}/%{name}-%{version}-%{release}-root-%(%{__id_u} -n)

BuildRoot:%{buildroot}
Summary:Mysql.rmp package of the company
License:GPL
Name:%{name}
Version:%{version}
Release:%{release}
Vendor:Mysql package of the company
Url:http://www.company.com/
SOURCE0:%{name}-%{version}.tar.gz
#SOURCE1:my3306.cnf
BuildRequires:ncurses-devel
Group:Application/Databases


Buildroot 定义了rpm包安装后的根目录，相当于咱们编译安装的 /目录，rpm打包就打包这部分；
下面部分需要注意：
BuildRoot:%{buildroot}　　　　　　　　　　　　　　//编译安装的根目录
SOURCE0:%{name}-%{version}.tar.gz　　　　　　//tar包
SOURCE1:my3306.cnf　　　　　　　　　　　　　　　//配置文件
BuildRequires:ncurses-devel　　                            //mysql编译安装时需要的环境包





　　%prep                      包括解压命令和其他一些shell脚本
　　%setup –q                解压tar包到BUILD目录
　　./configure
      Make
　　%setup                     安装，安装到BuildRoot（即${RPM_BUILD_ROOT}）目录下
　　%clean                      清理一些编译过程的目录
　　%file                         需要打包的文件目录
复制代码

#--- 该包的文字性描述
%description
The rpm package for the company


23 %install
24 if
25     getent group %mygrp >/dev/null
26 then
27     : OK group %mygrp already present
28 else
29     /usr/sbin/groupadd -r %mygrp 2>/dev/null || :
30 fi
31 if
32     id %myusr > /dev/null 2>&1
33 then
34     : OK user %myusr already present
35 else
36     /usr/sbin/useradd  -g %mygrp -s /sbin/nologin -c "User for Mysql" -d /var/tmp %myusr 2> /dev/null || :
37 fi
38 if ! [ -d %{buildroot}/mysqldata ]
39 then
40 %{__mkdir} -p %{buildroot}/mysqldata
41 fi
42
43 make install  DESTDIR=%{buildroot} INSTALLDIRS=vendor
44 install -m 755 %{SOURCE1} %{buildroot}/mysqldata/mysql/data/mysql3306/my3306_master.cnf
45 #install
46 %{__rm} -rf %{buildroot}/usr/local/mysql/sql-bench
47 %{__rm} -rf %{buildroot}/usr/local/mysql/mysql-test
48
49
50 %clean
51 %post
52
53 if
54     getent group %mygrp >/dev/null
55 then
56     : OK group %mygrp already present
57 else
58     /usr/sbin/groupadd -r %mygrp 2>/dev/null || :
59 fi
60 if
61     id %myusr > /dev/null 2>&1
62 then
63     : OK user %myusr already present
64 else
65     /usr/sbin/useradd  -g %mygrp -s /sbin/nologin -c "User for Mysql" -d /var/tmp %myusr 2> /dev/null || :
66 fi
67 /usr/local/mysql/bin/mysql_install_db --user=mysql --datadir=/mysqldata/mysql/data/mysql3306
68 chown mysql:mysql /mysqldata -R
-->


{% highlight text %}
### 0.define section                ← ###自定义宏段(可选)，方便后续引用
%define mysql_user mysql                            ← 定义宏，方便后续引用
%define _topdir /home/jinyang/databases/rpm-maker

### 1.The introduction section      ← ###介绍区域段
Name:           mysql                               ← 包名称，通常会在指定源码压缩包时引用
Version:        5.7.17                              ← 版本号，同上
Release:        1%{?dist}                           ← 释出号，每次制作rpm包时递增该数字
Summary:        MySQL from FooBar.                  ← 软件包简介，最好不要超过50字符
License:        GPLv2+ and BSD                      ← 许可，GPL、BSD、MIT等，也可以使用or

Group:          Applications/Databases              ← 程序组名，从/usr/share/doc/rpm-VER/GROUPS选择
URL:            http://kidding.com                  ← 一般为官网
Source0:        %{name}-boost-%{version}.tar.gz     ← 使用的源码包名称，可以用SourceN指定多个，如配置文件
#Patch0:         some-bugs.patch                    ← 如果需要打补丁，则依次填写
BuildRequires:  gcc,make                            ← 制作过程中用到的软件包
Requires:       pcre,pcre-devel,openssl,chkconfig   ← 软件运行所需软件包，可指定最低版本，如bash>=1.1.1

BuildRoot:      %_topdir/BUILDROOT                  ← 会打包该目录下文件，可查看安装后文件路径
Packager:       FooBar <foobar@kidding.com>
Vendor:         kidding.com

%description                        ← ###软件包的详细描述，可以撒丫子写了
It is a MySQL from FooBar.

#--- 2.The Prep section             ← ###准备阶段，主要是解压源码，并切换到源码目录下
%prep
%setup -q                                           ← 宏的作用是解压并切换到目录
#%patch0 -p1                                        ← 如果需要打补丁，则依次写

#--- 3.The Build Section            ← ###编译制作阶段，主要目的就是编译
%build

make %{?_smp_mflags}                                 ← 多核则并行编译

#--- 4.Install section              ← ###安装阶段
 %install
if [-d %{buildroot}]; then
   rm -rf %{buildroot}                               ← 清空下安装目录，实际会自动清除
fi
make install DESTDIR=%{buildroot}                    ← 安装到buildroot目录下
{% endhighlight %}

<!--
###  4.1 scripts section #没必要可以不写 %pre        #rpm安装前制行的脚本 if [ $1 == 1 ];then    #$1==1 代表的是第一次安装，2代表是升级，0代表是卸载         /usr/sbin/useradd -r nginx 2> /dev/null  ##其实这个脚本写的不完整fi %post       #安装后执行的脚本  %preun      #卸载前执行的脚本 if [ $1 == 0 ];then         /usr/sbin/userdel -r nginx 2> /dev/null fi %postun     #卸载后执行的脚本  ###  5.clean section 清理段,删除buildroot  %clean rm -rf %{buildroot}      ###  6.file section 要包含的文件 %files  %defattr (-,root,root,0755)   #设定默认权限，如果下面没有指定权限，则继承默认 /etc/           #下面的内容要根据你在%{rootbuild}下生成的来写     /usr/ /var/      ###  7.chagelog section  改变日志段 %changelog *  Fri Dec 29 2012 laoguang <ibuler@qq.com> - 1.0.14-1 - Initial version

{% highlight text %}
%global mysqldatadir /var/lib/mysql
%define _topdir /home/jinyang/databases/rpm-maker
Name:           mysql
Version:        5.7.17
Release:        1%{?dist}
Summary:        MySQL from FooBar.

Group:          Applications/Databases
License:        GPLv2+ and BSD
URL:            http://kidding.com
Source0:        %{name}-boost-%{version}.tar.gz
#BuildRequires:
#Requires:

%description
It is a MySQL from FooBar.

%prep
#%setup -q -T -D

%build
#(
#    if ! [ -d release ]; then
#        mkdir release
#    fi
#    pushd release
#    cmake ../%{name}-%{version}                  \
#        -DCMAKE_INSTALL_PREFIX=/opt/mysql-5.7      \
#        -DCMAKE_BUILD_TYPE=Debug                   \
#        -DWITH_EMBEDDED_SERVER=OFF                 \
#        -DWITH_EXAMPLE_STORAGE_ENGINE=ON           \
#        -DWITH_SAFEMALLOC=OFF                      \
#        -DWITH_BOOST=../%{name}-%{version}/boost
#    make %{?_smp_mflags}
#    popd
#)

%install
### Cleanup
##if [-d %{buildroot}]; then
##   rm -rf %{buildroot}
##fi
{% endhighlight %}


-->



<!--
%prep
%setup -q -T -a 0 -a 7 -a 10 -c -n %{src_dir}
参数列表：
    -T 禁止自动解压源码文件
    -D 解压前不删除目录
    -a 切换目录前，解压指定Source文件，例如-a 0表示解压Source0
    -b 切换目录后，解压指定Source文件，例如-a 0表示解压Source0
    -n 解压后目录名称与RPM名称不同，则通过该参数指定切换目录
-q
-c


http://laoguang.blog.51cto.com/6013350/1103628
-->


## 常用设置

接下来，看看一些常用的实用技巧。

### 配置脚本

很多制作 RPM 包的操作都是通过宏定义设置的，如下简单列举一下常见的宏定义操作。

{% highlight text %}
__os_install_post
    安装完之后的操作，例如去除二进制文件中的注释等；
__spec_install_pre
    在安装前的操作；
{% endhighlight %}

如果想取消某些操作，可以将这些操作设置为 ```%{nil}``` 即可。

{% highlight text %}
----- 查看对应的命令
$ rpm --showrc | grep -A 4 ': __os_install_post'
-14: __os_install_post
    /usr/lib/rpm/redhat/brp-compress
    %{!?__debug_package:
    /usr/lib/rpm/redhat/brp-strip %{__strip}
    /usr/lib/rpm/redhat/brp-strip-comment-note %{__strip} %{__objdump}

----- 在SPEC文件头部添加如下内容
%global __os_install_post %{nil}

----- 在用户的~/.rpmmacros文件中添加如下配置
%__os_install_post %{nil}
{% endhighlight %}

<!--
或者在全局配置文件中添加如下配置
/etc/rpm/macros
%__os_install_post %{nil}       
-->

### define vs. global

两者都可以用来进行变量定义，不过在细节上有些许差别，简单列举如下：

* define 用来定义宏，global 用来定义变量；
* 如果定义带参数的宏 (类似于函数)，必须要使用 define；
* 在 ```%{}``` 内部，必须要使用 global 而非 define；
* define 在使用时计算其值，而 global 则在定义时就计算其值；

可以简单参考如下的示例。

{% highlight spec %}
#--- %prep之前的参数是必须要有的
Name:           mysql
Version:        5.7.17
Release:        1%{?dist}
Summary:        MySQL from FooBar.
License:        GPLv2+ and BSD

%description
It is a MySQL from FooBar.

%prep
#--- 带参数时，必须使用%define定义
%define myecho() echo %1 %2
%{!?bar: %define bar defined}

echo 1: %{bar}
%{myecho 2: %{bar}}
echo 3: %{bar}

# 如下是输出内容
#1: defined
#2: defined
#3: %{bar}
{% endhighlight %}

显然 3 的输出是不符合预期的，可以将 ```%define``` 修改为 ```global``` 即可。

### 变量使用

define 定义的变量类似于局部变量，只在 ```%{!?foo: ... }``` 区间有效，不过 SPEC 并不会自动清除该变量，只有再次遇到 ```%{}``` 时才会清除，WTF!!!

在命令行中通过 ```--define 'with_ssl bundled'``` 进行定义，在 SPEC 脚本中，使用 ```%{?with_ssl: }``` 处理上述参数，或者通过 ```%{!?with_ssl: }``` 处理未定义参数时的情况。

{% highlight text %}
#--- 根据传入的变量，设置好相应的ssl_option变量
%{?with_ssl: %global ssl_option -DWITH_SSL=%{with_ssl}}

#--- 接下来，通过如下方式使用上述定义的变量
%{?ssl_option}
{% endhighlight %}

### 其它

如下是 ```%if``` 判断的使用。

{% highlight text %}
%if 0%{?rhel} == 6
%global compatver             5.1.17
BuildRequires:  systemd
%else
%global compatver             5.1.17
BuildRequires:  sysv
%endif
{% endhighlight %}


## 参考

很多关于 RPM 的介绍可以参考 [Maximum RPM](http://rpm.org/max-rpm-snapshot/index.html)，可以参考下 MySQL 的示例 [mysql.spec](/reference/databases/mysql/mysql.spec) 。

<!--
http://fedoraproject.org/wiki/How_to_create_an_RPM_package
-->

{% highlight text %}
{% endhighlight %}

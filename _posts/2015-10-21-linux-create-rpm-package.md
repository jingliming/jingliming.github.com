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
   -bb 制作成二进制RPM包
   -bs 制作源码的RPM包
   -ba 源码和二进制两种形式的RPM包
   -bl 检测BUILDROOT没有包含到rpm包中的文件

   -bp 执行到pre
   -bc 执行到build段
   -bi 执行install段

   --quiet 默认会输出每次执行的shell命令，此时忽略
{% endhighlight %}

一般来说，执行的顺序为 rpmbuild -bp，再 -bc 再 -bi，如果没问题，rpmbuild -ba 生成 src 包与二进制包，使用时通过 ```rpm -ivh xxx.rpm;``` 以及 ```rpm -e xxx.rpm``` 进行安装卸载。

## SPEC 文件

接下来准备 spec 文件，也是核心的内容，该文件包括三部分：介绍部分，编译部分，files 部分。接下来，是一个简单的示例，可以看看到底是如何制作 RPM 包的。

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
Requires:       pcre,pcre-devel,openssl,chkconfig   ← 安装时所需软件包，可使用bash >= 1.1.1
Requires(pre):  test                                ← 指定不同阶段的依赖

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

make %{?_smp_mflags}                                ← 多核则并行编译

#--- 4.Install section              ← ###安装阶段
 %install
if [-d %{buildroot}]; then
   rm -rf %{buildroot}                              ← 清空下安装目录，实际会自动清除
fi
make install DESTDIR=%{buildroot}                   ← 安装到buildroot目录下

#--- 4.1 scripts section            ← ###没必要可以不填
%pre                                                ← 安装前执行的脚本
%post                                               ← 安装后执行的脚本
%preun                                              ← 卸载前执行的脚本
%postun                                             ← 卸载后执行的脚本

#--- 5. Clean section               ← ###清理段，可以通过--clean删除BUILD
%clean
rm -rf %{buildroot}

#--- 6. File section                ← ###打包时要包含的文件
%files
%defattr (-,root,root,0755)                         ← 设定默认权限
%config(noreplace) /etc/my.cnf                      ← 表明是配置文件，noplace表示替换文件
%doc %{src_dir}/Docs/ChangeLog                      ← 表明这个是文档
%attr(644, root, root) %{_mandir}/man8/mysqld.8*    ← 分别是权限，属主，属组
%attr(755, root, root) %{_sbindir}/mysqld

#--- 7. Chagelog section            ← ###记录SPEC的修改日志段
%changelog
* Fri Dec 29 2012 foobar <foobar@kidding.com> - 1.0.0-1
- Initial version
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


### %prep

{% highlight text %}
%setup -q -T -a 0 -a 7 -a 10 -c -n %{src_dir}
参数列表：
    -T 禁止自动解压源码文件
    -D 解压前不删除目录
    -a 切换目录前，解压指定Source文件，例如-a 0表示解压Source0
    -b 切换目录后，解压指定Source文件，例如-a 0表示解压Source0
    -n 解压后目录名称与RPM名称不同，则通过该参数指定切换目录
    -c 解压缩之前先生成目录
{% endhighlight %}

<!--
%patch 最简单的补丁方式，自动指定patch level。
%patch 0 使用第0个补丁文件，相当于%patch ?p 0。
%patch -s 不显示打补丁时的信息。
%patch -T 将所有打补丁时产生的输出文件删除。
-->


## 常用设置

接下来，看看一些常用的实用技巧。

### 配置脚本

很多制作 RPM 包的操作都是通过宏定义设置的，如下简单列举一下常见的宏定义操作。

{% highlight text %}
__os_install_post
    安装完之后的操作，例如去除二进制文件中的注释等；
__spec_install_pre
    在安装前的操作，设置一些环境变量，然后删除BUILDROOT中的文件(如果文件多时耗时增加)；
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

可以使用 Requires(pre)、Requires(post) 等都是针对不同阶段的依赖设置；可以通过 ```%package PACKAGE-NAME``` 设置生成不同的 RPM 分支包。

另外，可以生成 GPG 签名，在此不再赘述。

#### RPM包查看

对于生成的 RPM 包，只能查看头部信息和脚本内容，指令分别如下。

{% highlight text %}
$ rpm --info -qp XXX.rpm
$ rpm --scripts -qp XXX.rpm
{% endhighlight %}


### 测试脚本

如下是一个测试用的脚本，可以用来生成简单的测试 SPEC 脚本，并执行。

该脚本会将 /tmp/foobar 目录作为工作目录，然后生成一个简单的 Hello world 程序。

{% highlight bash %}
#!/bin/bash

WORKSPACE=/tmp/foobar

echo "0. Prepare dirs"
mkdir -pv ${WORKSPACE}/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS,Workspace/foobar-1.0.0}

echo "1. Generate tar"
cd ${WORKSPACE}/Workspace/foobar-1.0.0
cat << EOF > main.c
#include
int main(int argc, char *argv)
{
    printf("Hello World!!!\n");
        return 0;
}
EOF
cat << "EOF" > Makefile
all: main.c
        gcc -o foobar $< -Wall
install:
        install -m 755 foobar ${DESTDIR}/usr/bin/foobar
EOF
cd ${WORKSPACE}/Workspace
tar -jcf foobar-1.0.0.tar.bz2 foobar-1.0.0
#[ "${WORKSPACE}/BUILD" != "/" ] && rm -rf "${WORKSPACE}/BUILD/"
rm -rf "${WORKSPACE}/SOURCES/foobar-1.0.0.tar.bz2"
mv foobar-1.0.0.tar.bz2 ${WORKSPACE}/SOURCES/

echo "2. Prepare spec-file"
cd ${WORKSPACE}
cat << EOF > SPECS/foobar-1.0.0.spec
### 0. The define section
%global _topdir ${WORKSPACE}

### 1.The introduction section
Name:           foobar
Version:        1.0.0
Release:        1%{?dist}
Summary:        Just a RPM example
License:        GPLv2+ and BSD
BuildRequires:  gcc,make
Source0:        %{name}-%{version}.tar.bz2
BuildRoot:      %_topdir/BUILDROOT

%description
It is a RPM example.

#--- 2.The Prep section
%prep
%setup -q
#%patch0 -p1

#--- 3.The Build Section
%build
make %{?_smp_mflags}
#echo %{_sysconfdir}
~
%install
#/usr/bin/
install -d -m 0751 %{buildroot}/%{_bindir}
make install DESTDIR=%{buildroot}
#--- 4.1 scripts section
%pre
echo "pre" >> /tmp/foobar
%post
echo "post" >> /tmp/foobar
%preun
echo "preun" >> /tmp/foobar
%postun
echo "postun" >> /tmp/foobar

#--- 5.clean section
%clean
rm -rf %{buildroot}

#--- 6.file section
%files
%defattr(-,root,root,-)
%attr(755, root, root) %{_bindir}/foobar

#--- 7.chagelog section
%changelog
* Fri Dec 29 2012 foobar foobar@kidding.com - 1.0.0-1
- Initial version
EOF

echo "3. Perform rpmbuild"
cd ${WORKSPACE}
rpmbuild --clean --define '_topdir /tmp/foobar' -ba SPECS/foobar-1.0.0.spec~
{% endhighlight %}



## 参考

很多关于 RPM 的介绍可以参考文档 [Maximum RPM](http://rpm.org/max-rpm-snapshot/index.html)；另外，还可以参考下 MySQL 源码包中的相关示例 [mysql.spec](/reference/databases/mysql/mysql.spec) 。

<!--
http://fedoraproject.org/wiki/How_to_create_an_RPM_package

包含生成GPG签名
http://laoguang.blog.51cto.com/6013350/1103628
-->

{% highlight text %}
{% endhighlight %}

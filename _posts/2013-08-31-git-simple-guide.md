---
title: Git 简明教程
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: git
description: 如果你严肃对待编程，就必定会使用"版本管理系统"（Version Control System）。
---

如果你严肃对待编程，就必定会使用"版本管理系统"（Version Control System）。

<!-- more -->

![git logo]({{ site.url }}/images/misc/git-logo.png "git logo"){: .pull-center width="60%" }

## 简介

不像 SVN ，Git 采用的是分布式的管理，简单来说，如下所示。

![git decentralized model]({{ site.url }}/images/misc/git-decentralized-model.png "git decentralized model"){: .pull-center width="60%" }

所有人除了可以从 origin 获取提交最新的代码之外，每个人还可以从不同人的代码库中获取代码，这也就意味着，比如两或多个人可以共同开发一个特性，测试无误之后再提交到 origin 即可。

另外，需要注意的是，所有版本控制系统，只能跟踪文本文件的改动；对于像图片、视频这类的二进制文件，虽能管理，但没法跟踪文件内容的变化，只知道从 1K 变为了 2K 而改变的内容不知道。


### 常见概念

简单介绍下版本开发过程中，经常使用的一些概念。

##### alpha、beta、gamma

用来标识测试的阶段和范围。

* alpha 内测，指开发团队内部测试的版本或者有限用户体验测试版本。
* beta 公测，即针对所有用户公开的测试版本。
* gamma，基本是 beta 做过一些修改后，成为正式发布的候选版本，也称为 RC 。

##### Release Candidate, RC

发行候选版本，和 Beta 版最大的差别在于 Beta 阶段会一直加入新的功能，但是到了 RC 版本，几乎就不会加入新的功能了，而主要着重于除错。

##### General Availability, GA

开发团队认为该版本是稳定版，有的软件可能会标识为 stable 或者 production 版，其意思和 GA 基本相同，也就是官方开始推荐广泛使用了。

##### 包命名

推荐按照 RPM 包的命名方式，也就是 packagename-version-release.arch.rpm 。

* name: 表示包的名称，包括主包名和分包名；
* version: 表示包的版本信息；
* release: 用于标识 rpm 包本身的发行号，可还包含适应的操作系统；
* arch: 表示主机平台，noarch 表示此包能安装到所有平台上面。

如 gd-devel-2.0.35-11.el6.x86_64.rpm ，gd 是这个包的主包名；devel 是这个包的分包名； 2.0.35 是表示版本信息，2 为主版本号，0 表示次版本号，35 为源码包的发行号也叫修订号； 11.el6 中的 11 是 rpm 的发行号， el6 表示 RHEL6； x86_64 是表示适合的平台。


### 常见操作

接下来，先看下一些常见的操作。

{% highlight text %}
----- 1. 创建新仓库
$ git init                             ← 会在.git目录下保存元数据

----- 2. 创建文件，并添加到版本库中
$ echo 'Hello World!' > README         ← 新建文件
$ git add README                       ← 添加到版本库中
$ git ls-files                         ← 查看版本库中的文件
$ git commit -m 'Initial Commit'       ← 提交并添加注释
$ git status                           ← 是否有未提交的文件

----- 3. 修改文件
$ echo 'What A Hell.' >> README        ← 修改文件添加一行
$ git commit -a -m 'Add a new line'    ← 提交下
$ git log --pretty=oneline             ← 查看提交信息

----- 4. 回退
$ echo 'It is A real world' > README   ← 覆盖之前的内容
$ git reset --hard HEAD                ← 回退到之前提交版本，HEAD^(上个) HEAD~N(上N个)
$ git reflog                           ← 查看版本号，貌似是UUID
$ git reset --hard 6fcfc89             ← 回退到指定的版本

----- 4. clone一个副本
$ git clone /tmp/foobar .              ← 从本地复制一个副本
{% endhighlight %}

<!--
如果是远端服务器上的仓库，你的命令会是这个样子：
git clone username@host:/path/to/repository
-->



### 工作区和版本库

**工作区** 就是在电脑上看到的目录以及文件 (.git隐藏目录版本库除外)，包括以后需要再新建的目录文件等等都属于工作区范畴。

**版本库 (Repository)** 工作区有一个隐藏目录 .git 就是版本库，其中比较重要的就是 stage(暂存区)，还有自动创建了第一个分支 master，以及指向 master 的一个指针 HEAD。

![git stage commit]({{ site.url }}/images/misc/git-stage-commit.png "git stage commit"){: .pull-center width="50%" }

如前所述，提交到版本库包含了两步：

1. 使用 git add 把文件添加到 stage；

2. 使用 git commit 把暂存区的所有内容提交到当前分支上。

关于其中的状态，可以在修改文件后、执行 add 后、执行 commit 后分别通过 status 命令查看文件的不同状态。

也就是说，添加到 stage 区域后，只要未提交，可以多次修改文件并 add 到 stage 区。

### 配置文件

总共有三个配置文件：/etc/gitconfig、~/.gitconfig、.git/config ，其优先级依次递增，后者的配置会覆盖前面的配置项。

#### 全局配置

简单来说就是 git config \-\-system 命令，添加了一个 system 参数，配置内容保存在 /etc/gitconfig 文件中，如下是设置 st 简化命令。

{% highlight text %}
$ git config --system alias.st status     # git st
{% endhighlight %}

#### 用户配置

执行 git config 会修改 ~/.gitconfig 文件的内容。

{% highlight text %}
----- 设置用户的默认用户名和密码
$ git config --global user.name "Your Name"
$ git config --global user.email you@example.com
$ git commit --amend --reset-author          # 未提交远端的可以进行修复

----- 开启颜色显示
$ git config --global color.ui true
{% endhighlight %}

#### 工作目录配置

进入工作根目录，运行 git config -e，这样就只会修改工作区的 .git/config 文件。


## 常见命令参考

在 clone 时会自动在本地分支和远程分支之间，建立一种追踪关系 (tracking)；例如，在通过 git clone 从远程库中复制代码库时，会自动将本地的 master 分支与 origin/master 分支对应。

当然，Git 也允许手动建立追踪关系，如下命令指定本地 master 分支追踪远程的 origin/next 分支。

{% highlight text %}
$ git branch --set-upstream master origin/next
{% endhighlight %}

### git pull

取回远程主机某个分支的更新，然后再与本地的指定分支合并，其完整格式以及常见命令如下。

{% highlight text %}
----- 完整命令
$ git pull <remote-host> <remote-branch>:<local-branch>

----- 取回origin主机的next分支，并与本地的master分支合并
$ git pull origin next:master        ← origin配置在.git/config文件中

----- 如果远程分支是与当前分支合并，则冒号后面的部分可以省略
$ git pull origin next               ← 等同于如下的两个命令
$ git fetch origin
$ git merge origin/next

----- 如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名
$ git pull origin

----- 如果当前分支只有一个追踪分支，连远程主机名都可以省略。
$ git pull
{% endhighlight %}

<!--
如果合并需要采用rebase模式，可以使用--rebase选项。
$ git pull --rebase <远程主机名> <远程分支名>:<本地分支名>
-->

### git push

该命令用于将本地分支的更新，推送到远程主机，其格式与 git pull 命令相仿，只是分支方向相反。

{% highlight text %}
----- 完整命令
$ git push <remote-host> <local-branch>:<remote-branch>

----- 将本地的master分支推送到origin主机的master分支；后者不存在，则会被新建
$ git push origin master

----- 省略本地分支名，则表示删除指定的远程分支
$ git push origin :master             ← 删除origin主机的master分支，等同于
$ git push origin --delete master

----- 如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略
$ git push origin                     ← 将当前分支推送到origin主机的对应分支

----- 如果当前分支只有一个追踪分支，那么主机名都可以省略。
$ git push

{% endhighlight %}

如果当前分支与多个主机存在追踪关系，则可以使用 -u 选项指定一个默认主机，这样后面就可以不加任何参数使用 git push ，命令如下。

{% highlight text %}
$ git push -u origin master
{% endhighlight %}

上面命令将本地的 master 分支推送到 origin 主机，同时指定 origin 为默认主机，后面就可以不加任何参数使用 git push 了。

#### simple & matching

简单来说，对于不带任何参数的 git push 命令；如果只推送当前分支，则称之为 simple方式；如果推送所有远程分支的对应本地分支，则为 matching，可以通过如下方式设置。

{% highlight text %}
$ git config --global push.default matching
$ git config --global push.default simple
{% endhighlight %}

另外，还有一种情况，就是不管是否存在对应的远程分支，都将本地的所有分支都推送到远程主机，这时就需要使用 \-\-all 选项。

{% highlight text %}
$ git push --all origin
{% endhighlight %}

上面命令表示，将所有本地分支都推送到 origin 主机。

### 其它

{% highlight text %}
$ git rev-parse --abbrev-ref HEAD            ← 当前版本

$ git show <HASHID>                          ← 某次提交的文件具体修改内容
$ git show <HASHID> file                     ← 某次某个文件的修改内容
$ git show <HASHID> --stat                   ← 修改了那些文件的统计
$ git diff --name-status HEAD~2 HEAD~3       ← 标记修改了的文件
$ git log -p <FILENAME>                      ← 某个文件修改历史
{% endhighlight %}

#### 更新单个文件

可以通过如下命令更新单个文件。

{% highlight text %}
----- 获取最新的版本，不过不会添加到工作区
$ git fetch

----- 通过下载的最新版本更新单个文件
$ git checkout origin/master -- path/to/file
{% endhighlight %}


<!--
如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做git pull合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用–force选项。

$ git push --force origin

上面命令使用–force选项，结果导致在远程主机产生一个”非直进式”的合并(non-fast-forward merge)。除非你很确定要这样做，否则应该尽量避免使用–force选项。

最后，git push不会推送标签(tag)，除非使用–tags选项。

$ git push origin --tags
-->




<!--
### 彻底删除文件

假设不小把一个大文件或者密码等信息不小心添加到了版本库，我们希望把它从版本库中永久删除不留痕迹，不仅要让它在版本历史里看不出来，还要把它占用的空间也释放出来。

简单来说，可以通过 git filter-branch 命令永久删除一些不需要的信息，对应了 git-filter-branch.sh 脚本，可以参考相应的参数 。

通过如下的示例参考。

{% highlight text %}
$ mkdir /tmp/test && cd /tmp/test
$ git init
$ dd if=/dev/urandom of=test.txt bs=10240 count=1024
$ git add test.txt
$ git commit -m "Init commit"
$ git rm test.txt
$ git commit -m "After 'git rm' command"

$ du -hs

$ git filter-branch --tree-filter 'rm -f test.txt' HEAD

Rewrite bb383961a2d13e12d92be5f5e5d37491a90dee66 (2/2)
Ref 'refs/heads/master'
 was rewritten
$ git ls-remote .
230b8d53e2a6d5669165eed55579b64dccd78d11        HEAD
230b8d53e2a6d5669165eed55579b64dccd78d11        refs/heads/master
bb383961a2d13e12d92be5f5e5d37491a90dee66        refs/original/refs/heads/master
$ git update-ref -d refs/original/refs/heads/master [bb383961a2d13e12d92be5f5e5d37491a90dee66]
$ git ls-remote .
230b8d53e2a6d5669165eed55579b64dccd78d11        HEAD
230b8d53e2a6d5669165eed55579b64dccd78d11        refs/heads/master
$ rm -rf .git/logs
$ git reflog --all
$ git prune
$ git gc
$ du -hs
 84K    .
{% endhighlight %}


折腾了半天，还是无法上传，于是，整个命令出来了：

    --tree-filter表示修改文件列表。
    --msg-filter表示修改提交信息，原提交信息从标准输入读入，新提交信息输出到标准输出。
    --prune-empty表示如果修改后的提交为空则扔掉不要。在一次试运行中我发现虽然文件被删除了，但是还剩下个空的提交，就查了下 man 文档，找到了这个选项。
    -f是忽略备份。不加这个选项第二次运行这个命令时会出错，意思是 git 上次做了备份，现在再要运行的话得处理掉上次的备份。
    --all是针对所有的分支。

试运行了几次，看到 40 多次提交逐一被重写，然后检查下，发现要删除的文件确实被删除了。于是高兴地到 github 建立新仓库，并上传了。


OK，这个文件已经完完全全删掉了，版本库已经不再占用空间了。
-->




## GitHub

关于使用 github 相关的内容。

### 1. 添加公钥(可选)

首先，生成一对密钥对，其中公钥需要添加到 Github 中，而私钥必须要保存好。

{% highlight text %}
----- 生成一对密钥，建议输入保护私钥密码
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "jinyang.sia@gmail.com"
{% endhighlight %}

按照如下步骤，将公钥添加到 Github 中。

![git github add sshkey]({{ site.url }}/images/misc/git-github-add-sshkey.png "git github add sshkey"){: .pull-center width="95%" }

### 2. 新建远程库

这个很简单，只需要添加个库名称，然后点击新建按钮即可。

![git github add repository]({{ site.url }}/images/misc/git-github-add-repository.png "git github add repository"){: .pull-center width="95%" }

实际上，新建完成后，会有提示如何导入一个库。

{% highlight text %}
----- 添加远端并提交
$ git remote add origin https://github.com/Jin-Yang/foobar.git
$ git push -u origin master             # 第一次导入
$ git push origin master                # 后面提交
{% endhighlight %}

<!--
### 3. clone 代码库

{% highlight text %}
$ git clone https://github.com/influxdata/influxdb.git
{% endhighlight %}

### 4. 删除代码库


### git-submodule

{% highlight text %}
git submodule init
git submodule update
{% endhighlight %}
-->



## 参考

Windows 下的客户端可以参考 [git for windows](https://git-for-windows.github.io/) 。

一本不错介绍 Git 的资料 [Pro Git Book](http://git-scm.com/book/) 。

对于一种不错的 Git 分支管理模式，也即如上的介绍，可以参考 [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/) 或者 [本地文档](/reference/misc/A successful Git branching model.mht) 。

<!--

git-filter-branch.sh


https://github.com/git/git

http://www.worldhello.net/gotgithub/index.html
http://www.cnblogs.com/zhangjing230/archive/2012/05/09/2489745.html

http://www.cnblogs.com/ctrlzhang/p/5195079.html

https://www.oschina.net/news/70368/git-advanced-commands

http://blog.csdn.net/wirelessqa/article/details/20152353
-->

{% highlight text %}
{% endhighlight %}

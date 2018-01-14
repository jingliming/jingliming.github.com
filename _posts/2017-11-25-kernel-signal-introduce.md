---
title: Linux 信号机制
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,signal,信号
description:
---


<!-- more -->

每个信号在 `signal.h` 头文件中通过宏进行定义，对于 CentOS 来说，实际是在 `/usr/include/asm-generic/signal.h` 中定义，对于编号以及信号名的映射关系可以通过 `kill -l` 命令查看。

其中，[1, 31] 是普通信号 而 [34, 64] 是实时信号。

## 信号处理

对于信号，通常有如下的几种处理方式：

1. 忽略。大部分信号都可以通过这种方式处理，不过 `SIGKILL` 和 `SIGSTOP` 两个信号有特殊用处，不能被忽略。
2. 默认动作。大多数信号的系统默认动作终止该进程。
3. 捕捉信号。也就是在收到信号时，执行一些用户自定义的函数。

其中信号可以简单通过 `signal()` 函数指定。


### 简单处理信号

其中 `signal()` 函数的声明如下：

{% highlight c %}
#include <signal.h>
typedef void(*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
{% endhighlight %}

其中，`signal()` 用于对该进程的某个特定信号 (signum) 注册一个相应的处理函数，也就是修改对该信号的默认处理动作。

注意，`signal()` 会堵塞当前正在处理的信号，不过不能阻塞其它信号，比如正在处理 `SIG_INT`，再来一个 `SIG_INT` 则会堵塞，但如果是 `SIG_QUIT` 则会被其中断，在处理完 `SIG_QUIT` 信号之后，`SIG_INT` 才会接着刚才处理。

{% highlight c %}
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void handler(int signum)
{
	printf("Got a signal %d\n", signum);
}

int main()
{
	signal(SIGINT, handler);

	while(1) {
		sleep(1);
		printf(".");
		fflush(stdout);
	}
	return 0;
}
{% endhighlight %}

也可以通过如下方式忽略某一个信号。

{% highlight c %}
signal(SIGHUP, SIG_IGN);
{% endhighlight %}

这里表示忽略 `SIGHUP` 这个信号，该信号与控制台有关，当控制台被关闭时，操作系统会向拥有控制台 SessionID 的所有进程发送 HUP 信号，而默认 HUP 信号的处理是退出程序。当远程登陆启动某个服务进程并在程序运行时关闭终端的话会导致服务进程退出，所以一般服务进程都会用 nohup 工具启动或写成一个 daemon。



<!--

### Signal VS. Sigaction

下面所指的signal都是指以前的older signal函数，现在大多系统都用sigaction重新实现了signal函数
1、signal在调用handler之前先把信号的handler指针恢复；sigaction调用之后不会恢复handler指针，直到再次调用sigaction修改handler指针。
：这样，(1)signal就会丢失信号，而且不能处理重复的信号，而sigaction就可以。因为signal在得到信号和调用handler之间有个时间把handler恢复了，这样再次接收到此信号就会执行默认的handler。（虽然有些调用，在handler的以开头再次置handler，这样只能保证丢信号的概率降低，但是不能保证所有的信号都能正确处理）
2、signal在调用过程不支持信号block；sigaction调用后在handler调用之前会把屏蔽信号（屏蔽信号中自动默认包含传送的该信号）加入信号中，handler调用后会自动恢复信号到原先的值。
（2）signal处理过程中就不能提供阻塞某些信号的功能，sigaction就可以阻指定的信号和本身处理的信号，直到handler处理结束。这样就可以阻塞本身处理的信号，到handler结束就可以再次接受重复的信号。
3、sigaction提供了比signal多的多的功能，可以参考man
-->










### SIGKILL VS. SIGSTOP

这两个信号比较特殊，无法在程序中进行屏蔽，用于一些特殊的用途。

#### SIGKILL

也就是直接的 `kill -9` 操作，为 root 提供了一种使进程强制终止方法，此时将会有操作系统直接回收该进程占用的资源，对于一些保存状态的应用就可能会导致异常。

#### SIGSTOP

对于前台运行的程序，可以通过 `Ctrl-Z` 终止程序，切换到后台，此时进程处于 `TASK_STOPPED` 状态，`ps` 命令显示处于 `T` 状态。如果要恢复运行，应该使用 `fg JOB-ID` 恢复运行，如果直接发送 `SIGCONT` 将会使进程退出。

可以参考 [WikiPedia SIGSTOP](http://en.wikipedia.org/wiki/SIGSTOP) 中的介绍，抄录如下：

{% highlight text %}
When SIGSTOP is sent to a process, the usual behaviour is to pause that process in its
current state. The process will only resume execution if it is sent the SIGCONT signal.
SIGSTOP and SIGCONT are used for job control in the Unix shell, among other purposes.
SIGSTOP cannot be caught or ignored.
{% endhighlight %}

也就是说，这个信号是用于 Shell 的任务管理，不能被用户屏蔽。其中常用的是 rsync 的同步任务，例如要清理一些空间，可以暂停运行，清理完成后重新启动运行。

{% highlight text %}
# kill -s STOP `pidof rsync`
# kill -s CONT `pidof rsync`
{% endhighlight %}



<!--
#define SIG_DFL ((void(*)(int))0)
#define SIG_IGN ((void(*)(int))1)
-->



















<!--
EINTR

## EAGAIN

实际上，在 Linux 中 EAGAIN 和 EWOULDBLOCK 相同。

以非阻塞方式打开文件或者 Sokcet 时，如果连续调用 read() 函数，而当没有数据可读时，又不会阻塞等待数据，那么此时就会返回 EAGAIN 错误，用来提示应用程序现在没有数据可读请稍后再试。

其它的，如当一个系统调用因为没有足够的资源而执行失败，返回 EAGAIN 提示其再调用一次，也许下次就能成功；例如 fork() 调用时可能内存不足。

## EINTR

当阻塞于某个系统调用时，如果进程捕获了某个信号，且相应信号处理函数返回后，该系统调用可能会返回一个 EINTR 错误。例如，在服务器端等待客户端链接，可能会获取到 EINTR 。

那么，为什么会有这个 EINTR 信号，如果出现了如何处理。首先，通常的处理方式是再次调用被中断的函数，而为什么有，还比较复杂一些。

假设有如下程序，主循环一直在读取数据，然后注册了一个中断函数用于标识退出程序，同时在退出前执行一些清理操作。

volatile int stop = 0;
void handler(int)
{
    stop++;
}

void event_loop (int sock)
{
    signal (SIGINT, handler);

    while (1) {
        if (stop) {
            printf ("do cleanup\n");
            return;
        }
        char buf [1];
        int rc = recv (sock, buf, 1, 0);
        if (rc == -1 && errno == EINTR)
            continue;
        printf ("perform an action\n");
    }
}

如上，当程序阻塞到 recv() 时，如果收到了 Ctrl-C 信号，那么在处理完之后实际上还会阻塞到 recv() 从而形成了死锁，如要要程序退出只能在接收到数据后进行处理，显然我们无法判断到底什么时候可能会收到数据。

通过返回的 EINTR 错误，让我们可以有机会进行处理，也就是如上的代码。

http://blog.csdn.net/hs794502825/article/details/52577622
http://www.cnblogs.com/Lynn-Zhang/p/5772403.html
http://www.cnblogs.com/Lynn-Zhang/p/5677118.html

http://blog.csdn.net/hzhsan/article/details/23650697
http://youbingchenyoubing.leanote.com/post/%E8%87%AA%E5%B7%B1%E8%B6%9F%E8%BF%87epoll%E7%9A%84%E5%9D%91
-->

{% highlight text %}
{% endhighlight %}

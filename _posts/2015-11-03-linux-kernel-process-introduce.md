---
title: Linux 进程简介
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,进程,daemon
description: 
---


<!-- more -->

## 守护进程 (daemon)

没有终端限制，让某个进程不因为用户、终端或者其他的变化而受到影响，那么就必须把这个进程变成一个守护进程。

守护进程的编程本身并不复杂，复杂的是各种版本的 Unix 的实现机制不尽相同，造成不同 Unix 环境下守护进程的编程规则并不一致。

守护进程的特性包括了：

1. 后台运行；
2. 与运行前的环境隔离开来，这些环境包括未关闭的文件描述符、控制终端、会话和进程组、工作目录以及文件创建 mask 等，这些环境通常是守护进程从执行它的父进程，特别是 shell，中继承下来的；
3. 守护进程的启动方式有其特殊之处，可以从启动脚本 `/etc/rc.d` 中启动，`crond` 启动，还可以由用户终端执行。

编程步骤包括了。

#### 1. 后台运行

创建子进程，父进程退出，在进程中调用 `fork()`，然后使父进程终止，让 Daemon 在子进程中后台执行，此时在形式上脱离了控制终端。

#### 2. 脱离控制终端、登录会话和进程组

进程属于一个进程组，进程组号 (GID) 就是进程组长的进程号；会话可以包含多个进程组，这些进程组共享一个控制终端；这个控制终端通常是创建进程的登录终端。

控制终端、登录会话和进程组通常是从父进程继承下来的，我们的目的就是要摆脱它们，使之不受它们的影响。在此通过调用 `setsid()` 创建新的会话+进程组，并使当前进程成为会话组长。

注意，当进程是会话组长时 `setsid()` 调用失败，但第一步已保证该进程不是会话组长。调用成功后，进程成为新的会话组长和新的进程组长，并与原来的登录会话和进程组脱离。由于会话过程对控制终端的独占性，进程同时与控制终端脱离。

#### 3. 禁止进程重新打开控制终端

现在，进程已经成为无终端的会话组长，但它可以重新申请打开一个控制终端，可以通过使进程不再成为会话组长来禁止进程重新打开控制终端。

一般来说，也就是再次 `fork()` 一次，不过这个是可选的。

#### 4. 关闭打开的文件描述符

进程从创建它的父进程那里继承了打开的文件描述符，如不关闭，将会浪费系统资源，造成进程所在的文件系统无法卸下以及引起无法预料的错误。

一般需要关闭 0~2 标准输出。

<!-- getdtablesize() 返回所在进程的文件描述符表的项数，即该进程打开的文件数目-->

#### 5. 改变当前工作目录

进程活动时，其工作目录所在的文件系统不能卸下，一般需要将工作目录改变到根目录。对于需要转储核心，写运行日志的进程将工作目录改变到特定目录如 `chdir("/tmp")` 。

#### 6.重设文件权限掩模

进程从创建它的父进程那里继承了文件权限掩模，它可能修改守护进程所创建的文件的存取位，为防止这一点，将文件创建掩模清除 `umask(0)`。

#### 7. 处理 SIGCHLD 信号

处理 `SIGCHLD` 信号并不是必须的，但对于某些进程，特别是服务器进程往往在请求到来时生成子进程处理请求。如果父进程不等待子进程结束，子进程将成为僵尸进程 (zombie) 从而占用系统资源。如果父进程等待子进程结束，将增加父进程的负担，影响服务器进程的并发性能。

在 Linux 下可以简单地将 `SIGCHLD` 信号的操作设为 `SIG_IGN，signal(SIGCHLD, SIG_IGN)`，这样，内核在子进程结束时不会产生僵尸进程，这一点与 BSD4 不同，BSD4 下必须显式等待子进程结束才能释放僵尸进程。

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/stat.h>

int main()
{
        FILE *fp;
        time_t t;

        int pid;
        int i;

        pid = fork();   // STEP 1
        if (pid < -1) {        // error
                perror("fork()");
                exit(EXIT_FAILURE);
        } else if (pid != 0) { // parent
                fprintf(stdout, "Parent PID(%d) running", pid);
                exit(EXIT_SUCCESS);
        }

        /* child */
        setsid();     // STEP 2, Detach from session.

        pid = fork(); // STEP 3, fork again to prevent recreate a console.
        if (pid < -1) {        // error
                perror("fork()");
                exit(EXIT_FAILURE);
        } else if (pid != 0) { // parent
                fprintf(stdout, "Parent PID(%d) running", pid);
                exit(EXIT_SUCCESS);
        }

        for(i = 0; i < getdtablesize(); i++) // STEP 4, close all opend file discript.
                close(i);

        chdir("/tmp");  // STEP 5
        umask(0);       // STEP 6

        while (1) {
                fp = fopen("test.log", "a");
                if(fp != NULL ) {
                        t = time(0);
                        fprintf(fp, "I'm here at %s\n", asctime(localtime(&t)) );
                        fclose(fp);
                }
                sleep(60);
        }
}
{% endhighlight %}




### 一次 VS. 两次 fork()

实际上，在上述的第二步中，可以再执行一次 `fork()` 操作。

第一次 `fork()` 后子进程继承了父进程的进程组 ID，但有一个新进程 ID，这就保证了子进程不是一个进程组的首进程，然后 `setsid()` 是为了跟主进程的 SID PGID 脱离设置成子进程的 SID PGID。

虽然此时子进程已经被 init 接管了，但是只有 `setsid()` 之后才算是跟那个主进程完全脱离，不受他的影响。

#### 第二次 fork()

第二次 `fork()` 不是必须的，主要目的是为了防止进程再次打开一个控制终端。

因为打开一个控制终端的前提条件是该进程必须是会话组长，那么再 `fork()` 一次后，子进程 ID 不再等于 sid (sid 是进程父进程的 sid)，所以也无法打开新的控制终端。

此时这个子进程是首进程了 ，然后此时为了避免他是首进程，所以又 `fork()` 了一次。


## 参考


{% highlight python %}
{% endhighlight %}

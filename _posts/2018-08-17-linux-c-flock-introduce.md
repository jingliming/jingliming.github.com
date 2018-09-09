---
title: Linux C Flock 使用
layout: post
comments: true
language: chinese
category: [linux,program]
keywords: linux,program,c,flock
description: 在某些场景下，例如需要保证单个进程运行，通常的做法是生成一个 PID 文件，并将当前的进程 PID 写入，每次进程启动时检查文件以及进程是否存在。如果进程异常崩溃没有删除文件，而 Linux 中 PID 可以复用，那么就可能会导致误认为进程存在，虽然概率很低。其实在 Linux 中可以通过 flock 实现。
---

在某些场景下，例如需要保证单个进程运行，通常的做法是生成一个 PID 文件，并将当前的进程 PID 写入，每次进程启动时检查文件以及进程是否存在。

如果进程异常崩溃没有删除文件，而 Linux 中 PID 可以复用，那么就可能会导致误认为进程存在，虽然概率很低。

其实在 Linux 中可以通过 flock 实现。

<!-- more -->

## Flock

在 Linux 中有个简单的实现，也就是 `flock()` ，这是一个建议性锁，不具备强制性。

也就是说，一个进程使用 flock 将文件锁住，另一个进程仍然可以操作正在被锁的文件，修改文件中的数据，这也就是所谓的建议性锁的内核处理策略。

{% highlight c %}
#include <sys/file.h>

int flock (intfd, int operation);
{% endhighlight %}

其中 `flock()` 主要有三种操作类型：

* LOCK_SH，共享锁，多个进程可以使用同一把锁，常被用作读共享锁；
* LOCK_EX，排他锁，同时只允许一个进程使用，常被用作写锁；
* LOCK_UN，释放锁；

默认的操作是阻塞，可以通过 `LOCK_NB` 设置为非阻塞。

### 脚本

另外，在命令行中，也可以通过类似如下的方式进行测试。

{% highlight text %}
$ flock -xn /tmp/foobar.lock -c "echo 'Hi world'"
{% endhighlight %}

常见的如 crontab 。


## PIDFile

默认进程在使用 flock 尝试锁文件时，如果文件已经被其它进程锁住，进程会被阻塞直到锁被释放掉。也可以使用 `LOCK_NB` 参数，此时如果被锁，那么会直接返回错误，对应的 `errno` 为 `EWOULDBLOCK`。

简单来说，也就是阻塞、非阻塞两种工作模式。

flock 可以通过 `LOCK_UN` 显示的释放锁，也可以直接通过关闭 fd 的来释放文件锁，这意味着 flock 会随着进程的关闭而被自动释放掉。

<!--
注意，`flock()` 可以被 `fork()` 继承。

也就是以排它非阻塞的方式运行。
http://blog.jobbole.com/102538/

#include <stdio.h>
#include <sys/file.h>

int check_pidfile(char *file)
{
        int fd, rc;

        if (file == NULL)
                return -1;

        fd = open(file, O_RDWR);
        if (fd < 0)
                return -2;

        rc = flock(fd, LOCK_EX | LOCK_NB);
        if (rc < 0)
                return -3;

        return 0;
}
-->


## 注意事项

在使用如下测试时，需要保证 `/tmp/foobar.txt` 存在。

### 同一进程

在同一个进程中可以多次进行加锁而不会阻塞，可以通过如下方式进行测试。

{% highlight c %}
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/file.h>

int main(void)
{
        int rc, fd;

        fd = open("/tmp/foobar.txt", O_RDWR);
        printf("current fd: %d\n", fd);
        rc = flock(fd, LOCK_EX);
        printf("get lock rc: %d\n", rc);
        rc = flock(fd, LOCK_EX);
        printf("get lock again, rc: %d\n", rc);

	sleep(1000);

	return 0;
}
{% endhighlight %}

当启动第二个程序时就会被阻塞掉。

### 文件描述符

`flock` 创建的锁是和文件打开表项 `struct file` 相关联的，而不是文件描述符 `fd`。

也就是通过 `fork()` 或者 `dup()` 复制 `fd` 后，可以通过这两个 `fd` 同时操作锁，但是关闭其中一个 `fd` 锁并不会释放，因为 `struct file` 并没有释放，只有关闭所有复制出的 `fd`，锁才会释放。

{% highlight c %}
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/file.h>

int main(void)
{
        int rc, fd1, fd2;

        fd1 = open("/tmp/foobar.txt", O_RDWR);
        fd2 = dup(fd1);
        close(fd1);

        printf("current fd: %d\n", fd2);
        rc = flock(fd2, LOCK_EX);
        printf("get lock2, ret: %d\n", rc);
        sleep(10);
        close(fd2);
        printf("release\n");

        sleep(10000);

        return 0;
}
{% endhighlight %}

如上，在关闭掉所有的文件描述符之后才会释放掉文件锁。

### 子进程

{% highlight c %}
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/file.h>

int main(void)
{
        int rc, pid, fd;

        fd = open("/tmp/foobar.txt", O_RDWR);

        pid = fork();
        if (pid == 0) { /* child */
                rc = flock(fd, LOCK_EX);
                printf("chile get lock, fd: %d, ret: %d\n", fd, rc);
                sleep(10);
                printf("child exit\n");
                exit(0);
        }

        rc = flock(fd, LOCK_EX);
        printf("parent get lock, fd: %d, ret: %d\n", fd, rc);
        sleep(12);
        printf("parent exit\n");

        return 0;
}
{% endhighlight %}

子进程持有锁，并不影响父进程通过相同的 fd 获取锁，反之亦然。

### 多次打开

当使用 `open()` 两次打开同一个文件，得到的两个 `fd` 是独立的，内核会使用两个 `struct file` 对象，通过其中一个加锁，通过另一个无法解锁，并且在前一个解锁前也无法上锁。

{% highlight c %}
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/file.h>

int main(void)
{
        int rc, fd1, fd2;

        fd1 = open("/tmp/foobar.txt", O_RDWR);
        fd2 = open("/tmp/foobar.txt", O_RDWR);
        printf("fd1: %d, fd2: %d\n", fd1, fd2);

        rc = flock(fd1, LOCK_EX);
        printf("get lock1, ret: %d\n", rc);
        close(fd1);

        rc = flock(fd2, LOCK_EX);
        printf("get lock2, ret: %d\n", rc);

        return 0;
}
{% endhighlight %}

上述代码中，如果注释掉 `close(fd1)` 会被阻塞。

<!--
(3) 使用exec后，文件锁的状态不变。
(4) flock不能再NFS文件系统上使用，如果要在NFS使用文件锁，请使用fcntl。
(5) flock锁可递归，即通过dup或者或者fork产生的两个fd，都可以加锁而不会产生死锁。

2. lockf与fcntl

函数原型
#include

int lockf(int fd, int cmd, off_t len);

fd为通过open返回的打开文件描述符。

cmd的取值为：

      F_LOCK：给文件互斥加锁，若文件以被加锁，则会一直阻塞到锁被释放。

      F_TLOCK：同F_LOCK，但若文件已被加锁，不会阻塞，而回返回错误。

      F_ULOCK：解锁。

      F_TEST：测试文件是否被上锁，若文件没被上锁则返回0，否则返回-1。

      len：为从文件当前位置的起始要锁住的长度。

通过函数参数的功能，可以看出lockf只支持排他锁，不支持共享锁。

?
1
2
3
4
5
6
7
8
9
10
11
12

#include
#include
int fcntl(int fd, int cmd, ... /* arg */ );
struct flock {
...
short l_type;/* Type of lock: F_RDLCK, F_WRLCK, F_UNLCK */
short l_whence; /* How to interpret l_start: SEEK_SET, SEEK_CUR, SEEK_END */
off_t l_start;  /* Starting offset for lock */
off_t l_len;   /* Number of bytes to lock */
pid_t l_pid; /* PID of process blocking our lock (F_GETLK only) */
...
  };

文件记录加锁相关的cmd 分三种：

       F_SETLK：申请锁（读锁F_RDLCK，写锁F_WRLCK）或者释放所（F_UNLCK），但是如果kernel无法将锁授予本进程（被其他进程抢了先，占了锁），不傻等，返回error。

       F_SETLKW：和F_SETLK几乎一样，唯一的区别，这厮是个死心眼的主儿，申请不到，就傻等。

       F_GETLK：这个接口是获取锁的相关信息： 这个接口会修改我们传入的struct flock。

通过函数参数功能可以看出fcntl是功能最强大的，它既支持共享锁又支持排他锁，即可以锁住整个文件，又能只锁文件的某一部分。

下面看fcntl/lockf的特性：

       (1) 上锁可递归，如果一个进程对一个文件区间已经有一把锁，后来进程又企图在同一区间再加一把锁，则新锁将替换老锁。

       (2) 加读锁（共享锁）文件必须是读打开的，加写锁（排他锁）文件必须是写打开。

       (3) 进程不能使用F_GETLK命令来测试它自己是否再文件的某一部分持有一把锁。F_GETLK命令定义说明，返回信息指示是否现存的锁阻止调用进程设置它自己的锁。因为，F_SETLK和F_SETLKW命令总是替换进程的现有锁，所以调用进程绝不会阻塞再自己持有的锁上，于是F_GETLK命令绝不会报告调用进程自己持有的锁。

       (4) 进程终止时，他所建立的所有文件锁都会被释放，队医flock也是一样的。

       (5) 任何时候关闭一个描述符时，则该进程通过这一描述符可以引用的文件上的任何一把锁都被释放（这些锁都是该进程设置的），这一点与flock不同。如：
?
1
2
3
4
5
6
7

fd1 = open(pathname, …);

lockf(fd1, F_LOCK, 0);

fd2 = dup(fd1);

close(fd2);

则在close(fd2)后，再fd1上设置的锁会被释放，如果将dup换为open，以打开另一描述符上的同一文件，则效果也一样。
?
1
2
3
4
5
6
7

fd1 = open(pathname, …);

lockf(fd1, F_LOCK, 0);

fd2 = open(pathname, …);

close(fd2);

        (6) 由fork产生的子进程不继承父进程所设置的锁，这点与flock也不同。

        (7) 在执行exec后，新程序可以继承原程序的锁，这点和flock是相同的。（如果对fd设置了close-on-exec，则exec前会关闭fd，相应文件的锁也会被释放）。

        (8) 支持强制性锁：对一个特定文件打开其设置组ID位(S_ISGID)，并关闭其组执行位(S_IXGRP)，则对该文件开启了强制性锁机制。再Linux中如果要使用强制性锁，则要在文件系统mount时，使用_omand打开该机制。

3. 两种锁的关系

那么flock和lockf/fcntl所上的锁有什么关系呢？答案时互不影响。测试程序如下：
?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31

#include <unistd.h>

#include <stdio.h>

#include <stdlib.h>

#include <sys/file.h>

int main(int argc, char **argv)

{

  int fd, ret;

  int pid;

  fd = open("./tmp.txt", O_RDWR);

  ret = flock(fd, LOCK_EX);

  printf("flock return ret : %dn", ret);

  ret = lockf(fd, F_LOCK, 0);

  printf("lockf return ret: %dn", ret);

  sleep(100);

  return 0;

}

测试结果如下：
?
1
2
3
4
5

$./a.out

flock return ret : 0

lockf return ret: 0

可见flock的加锁，并不影响lockf的加锁。另外我们可以通过/proc/locks查看进程获取锁的状态。

$ps aux | grep a.out | grep -v grep

123751   18849  0.0  0.0  11904   440 pts/5    S+   01:09   0:00 ./a.out

$sudo cat /proc/locks | grep 18849

1: POSIX  ADVISORY  WRITE 18849 08:02:852674 0 EOF

2: FLOCK  ADVISORY  WRITE 18849 08:02:852674 0 EOF

我们可以看到/proc/locks下面有锁的信息：我现在分别叙述下含义：

       1) POSIX FLOCK 这个比较明确，就是哪个类型的锁。flock系统调用产生的是FLOCK，fcntl调用F_SETLK，F_SETLKW或者lockf产生的是POSIX类型，有次可见两种调用产生的锁的类型是不同的；

       2) ADVISORY表明是劝告锁；

       3) WRITE顾名思义，是写锁，还有读锁；

       4) 18849是持有锁的进程ID。当然对于flock这种类型的锁，会出现进程已经退出的状况。

       5) 08:02:852674表示的对应磁盘文件的所在设备的主设备好，次设备号，还有文件对应的inode number。

       6) 0表示的是所的其实位置

       7) EOF表示的是结束位置。 这两个字段对fcntl类型比较有用，对flock来是总是0 和EOF。

以上就是linux中fcntl()、lockf和flock的区别的全部内容，希望本文对大家有所帮助。

-->





{% highlight text %}
{% endhighlight %}

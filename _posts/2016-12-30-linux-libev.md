---
title: libev 事件库
layout: post
comments: true
language: chinese
category: [linux,program]
keywords: linux,program,libev,event loop
description: libev 与 libevent 类似，是一个 C 语言编写的，高性能的事件循环库，在此详细介绍下。
---

libev 是一个 C 语言编写的，高性能的事件循环库，与此类似的事件循环库还有 libevent、libubox 等，在此详细介绍下 libev 相关的内容。

<!-- more -->

## 简介

关于 libev 详见官网 [http://software.schmorp.de](http://software.schmorp.de/pkg/libev.html)，其帮助文档可以参考 [官方文档](http://pod.tst.eu/http://cvs.schmorp.de/libev/ev.pod)，安装完之后，可通过 ```man 3 ev``` 查看帮助信息，文档也在源码中保存了一份，可以通过 ```man -l ev.3``` 命令查看。

### 安装

安装可以源码安装，或者在 CentOS 中，最简单可通过如下方式安装。

{% highlight text %}
----- 安装库
# yum install libev libev-devel
{% endhighlight %}

### 示例程序

如下是一个简单的示例程序。

{% highlight c %}
#include <ev.h>       // a single header file is required
#include <stdio.h>    // for puts

// every watcher type has its own typedef'd struct with the name ev_TYPE
ev_io stdin_watcher;
ev_timer timeout_watcher;

// all watcher callbacks have a similar signature
// this callback is called when data is readable on stdin
static void stdin_cb (EV_P_ ev_io *w, int revents)
{
    puts ("stdin ready");
    // for one-shot events, one must manually stop the watcher
    // with its corresponding stop function.
    ev_io_stop (EV_A_ w);

    // this causes all nested ev_run's to stop iterating
    ev_break (EV_A_ EVBREAK_ALL);
}

// another callback, this time for a time-out
static void timeout_cb (EV_P_ ev_timer *w, int revents)
{
    puts ("timeout");
    // this causes the innermost ev_run to stop iterating
    ev_break (EV_A_ EVBREAK_ONE);
}

int main (void)
{
    // use the default event loop unless you have special needs
    struct ev_loop *loop = EV_DEFAULT; /* OR ev_default_loop(0) */

    // initialise an io watcher, then start it
    // this one will watch for stdin to become readable
    ev_io_init (&stdin_watcher, stdin_cb, /*STDIN_FILENO*/ 0, EV_READ);
    ev_io_start (EV_A_ &stdin_watcher);

    // initialise a timer watcher, then start it
    // simple non-repeating 5.5 second timeout
    ev_timer_init (&timeout_watcher, timeout_cb, 5.5, 0.);
    ev_timer_start (EV_A_ &timeout_watcher);

    // now wait for events to arrive
    ev_run (loop, 0);

    // break was called, so exit
    return 0;
}
{% endhighlight %}

可以通过如下命令编译。

{% highlight text %}
----- 编译示例程序
$ gcc -lev example.c -o example
{% endhighlight %}

### 执行过程

libev 是一个事件循环，首先需要注册感兴趣的事件，libev 会监控这些事件，当事件发生时调用相应的处理函数，也就是回调函数。其处理过程为：

1. 初始化一个事件循环<br>可以通过 ev_default_loop(0) 或者 EV_DEFAULT 初始化，两者等价。
2. 定义数据<br>在 ev.h 中定义了各种类型的，如 ev_io、ev_timer、ev_signal等。
3. 注册感兴趣的事件<br>在此被称为 watchers ，这个是 C 结构体。
4. 启动监控<br>启动上步注册的事件，如 ev_io_start()、ev_timer_start() 等。
5. 启动 libev<br>重复 1,2 步，然后启动 libev 事件循环，直接执行 ev_run() 即可。

## 源码详解

libev 通过观察器 (watcher) 来监听各种事件，watcher 包括了事件类型、优先级、触发条件和回调函数等参数；将其注册到事件循环上，在满足注册的条件时，会触发观察器，调用它的回调函数。


{% highlight text %}
enum {
  EV_UNDEF    = (int)0xFFFFFFFF, /* guaranteed to be invalid */
  EV_NONE     =            0x00, /* no events */
  EV_READ     =            0x01, /* ev_io detected read will not block */
  EV_WRITE    =            0x02, /* ev_io detected write will not block */
  EV__IOFDSET =            0x80, /* internal use only */
  EV_IO       =         EV_READ, /* alias for type-detection */
  EV_TIMER    =      0x00000100, /* timer timed out */
#if EV_COMPAT3
  EV_TIMEOUT  =        EV_TIMER, /* pre 4.0 API compatibility */
#endif
  EV_PERIODIC =      0x00000200, /* periodic timer timed out */
  EV_SIGNAL   =      0x00000400, /* signal was received */
  EV_CHILD    =      0x00000800, /* child/pid had status change */
  EV_STAT     =      0x00001000, /* stat data changed */
  EV_IDLE     =      0x00002000, /* event loop is idling */
  EV_PREPARE  =      0x00004000, /* event loop about to poll */
  EV_CHECK    =      0x00008000, /* event loop finished poll */
  EV_EMBED    =      0x00010000, /* embedded event loop needs sweep */
  EV_FORK     =      0x00020000, /* event loop resumed in child */
  EV_CLEANUP  =      0x00040000, /* event loop resumed in child */
  EV_ASYNC    =      0x00080000, /* async intra-loop signal */
  EV_CUSTOM   =      0x01000000, /* for use by user code */
  EV_ERROR    = (int)0x80000000  /* sent when an error occurs */
{% endhighlight %}


libev 中的观察器分为 4 种状态：初始化、启动/活动、等待、停止。

首先需要对 watcher 初始化，可通过 ```ev_TYPE_init()``` 或者 ```ev_init()```+```ev_TYPE_set()``` 初始化，两者等效；实际就是设置对应结构体的初始值。

{% highlight c %}
#define ev_io_init(ev,cb,fd,events)              \
    do { ev_init ((ev), (cb)); ev_io_set ((ev),(fd),(events)); } while (0)
#define ev_timer_init(ev,cb,after,repeat)        \
    do { ev_init ((ev), (cb)); ev_timer_set ((ev),(after),(repeat)); } while (0)
#define ev_periodic_init(ev,cb,ofs,ival,rcb)     \
    do { ev_init ((ev), (cb)); ev_periodic_set ((ev),(ofs),(ival),(rcb)); } while (0)
#define ev_signal_init(ev,cb,signum)             \
    do { ev_init ((ev), (cb)); ev_signal_set ((ev), (signum)); } while (0)
#define ev_child_init(ev,cb,pid,trace)           \
    do { ev_init ((ev), (cb)); ev_child_set ((ev),(pid),(trace)); } while (0)
#define ev_stat_init(ev,cb,path,interval)        \
    do { ev_init ((ev), (cb)); ev_stat_set ((ev),(path),(interval)); } while (0)
#define ev_idle_init(ev,cb)                      \
    do { ev_init ((ev), (cb)); ev_idle_set ((ev)); } while (0)
#define ev_prepare_init(ev,cb)                   \
    do { ev_init ((ev), (cb)); ev_prepare_set ((ev)); } while (0)
#define ev_check_init(ev,cb)                     \
    do { ev_init ((ev), (cb)); ev_check_set ((ev)); } while (0)
#define ev_embed_init(ev,cb,other)               \
    do { ev_init ((ev), (cb)); ev_embed_set ((ev),(other)); } while (0)
#define ev_fork_init(ev,cb)                      \
    do { ev_init ((ev), (cb)); ev_fork_set ((ev)); } while (0)
#define ev_cleanup_init(ev,cb)                   \
    do { ev_init ((ev), (cb)); ev_cleanup_set ((ev)); } while (0)
#define ev_async_init(ev,cb)                     \
    do { ev_init ((ev), (cb)); ev_async_set ((ev)); } while (0)
{% endhighlight %}

接下来，通过 ```ev_TYPE_start()```、```ev_TYPE_stop()``` 来启动、停止观察器，停止同时会释放内存。

### 结构体

libev 通过 C 语言实现，其中通过宏实现了一种类似的继承机制，也就是其中各种 Watchers 的部分成员变量是相同的，只有少部分成员为各自独有，接下来简单介绍下。

每个 watcher 都会包含 EV_WATCHER 宏定义的内容，该宏实际会包含如下内容，其中 type 对应类型，如 ev_io 等。

{% highlight c %}
# define EV_CB_DECLARE(type) void (*cb)(EV_P_ struct type *w, int revents);

#define EV_WATCHER(type) \
  int active;          /* private，是否激活，通过start()/stop()处理 */ \
  int pending;         /* private，有事件就绪等待处理，对应了等待队列的下标 */ \
  EV_DECL_PRIORITY     /* private，定义优先级，如果没有使用优先级则是空 */ \
  EV_COMMON            /* rw，私有数据，一般是void *data */ \
  EV_CB_DECLARE (type) /* private，回调函数 */

#define EV_WATCHER_LIST(type)           \
  EV_WATCHER (type)                     \
  struct ev_watcher_list *next; /* private */

#define EV_WATCHER_TIME(type)           \
  EV_WATCHER (type)                     \
  ev_tstamp at;                 /* private */

typedef struct ev_watcher {
  EV_WATCHER (ev_watcher)
} ev_watcher;

typedef struct ev_watcher_list {
  EV_WATCHER_LIST (ev_watcher_list)
} ev_watcher_list;

typedef struct ev_io {
  EV_WATCHER_LIST (ev_io)

  int fd;     /* ro */
  int events; /* ro */
} ev_io;

typedef struct ev_timer {
  EV_WATCHER_TIME (ev_timer)

  ev_tstamp repeat; /* rw */
} ev_timer;
{% endhighlight %}

如上的 ev_watcher 结构体可以时为 “基类”，通过宏 EV_WATCHER 定义了它的所有成员；而像 IO Watcher、Signal Watcher 是以链表的形式进行组织的，所以在 ev_watcher 基类的基础上，定义了 ev_watcher 的子类 ev_watcher_list 。

#### 多实例支持

ev_loop 是主循环，保存了与循环相关的很多变量，而 ```EV_MULTIPLICITY``` 是一个条件编译的宏，表明是否支持有多个 ev_loop 实例存在，表现在源码中表示是否需要传递 ```struct ev_loop *loop``` 参数，一般来说，每个线程中有且仅有一个 ev_loop 实例。

<!--
如果支持多个loop，则default_loop_struct就是一个静态的struct ev_loop类型的结构体，其中包含了各种成员，比如ev_tstamp ev_rt_now;  int  pendingpri;等等。

ev_default_loop_ptr就是指向struct  ev_loop 类型的指针。

如果不支持多个loop，则上述的struct  ev_loop结构就不复存在，其成员都是以静态变量的形式进行定义，而ev_default_loop_ptr也只是一个int变量，用来表明”loop”是否已经初始化成功。
-->

### 系统时间

先看下 libev 是如何使用时间的，因为该库中很多与时间相关的操作；在 libev.m4 中，定义了与之相关的宏，如下所示。

{% highlight text %}
AC_CHECK_FUNCS(clock_gettime, [], [
   dnl on linux, try syscall wrapper first
   if test $(uname) = Linux; then
      AC_MSG_CHECKING(for clock_gettime syscall)
      AC_LINK_IFELSE([AC_LANG_PROGRAM(
         [#include <unistd.h>
          #include <sys/syscall.h>
          #include <time.h>],
         [struct timespec ts; int status = syscall (SYS_clock_gettime, CLOCK_REALTIME, &ts)])],
         [ac_have_clock_syscall=1
          AC_DEFINE(HAVE_CLOCK_SYSCALL, 1, Define to 1 to use the syscall interface for clock_gettime)
          AC_MSG_RESULT(yes)],
         [AC_MSG_RESULT(no)])
   fi
   if test -z "$LIBEV_M4_AVOID_LIBRT" && test -z "$ac_have_clock_syscall"; then
      AC_CHECK_LIB(rt, clock_gettime)
      unset ac_cv_func_clock_gettime
      AC_CHECK_FUNCS(clock_gettime)
   fi
])

AC_CHECK_FUNCS(nanosleep, [], [
   if test -z "$LIBEV_M4_AVOID_LIBRT"; then
      AC_CHECK_LIB(rt, nanosleep)
      unset ac_cv_func_nanosleep
      AC_CHECK_FUNCS(nanosleep)
   fi
])
{% endhighlight %}

首先会检测 ```clock_gettime()``` 系统调用是否可用，如果可用会定义 ```HAVE_CLOCK_SYSCALL``` 宏。

libev 提供了单调递增 (monotonic) 以及实时时间 (realtime) 两种记时方式，其宏定义的方式如下，而 ```HAVE_CLOCK_SYSCALL``` 和 ```HAVE_CLOCK_GETTIME``` 的详见 libev.m4 中定义，优先使用 ```SYS_clock_gettime()``` 系统调用 API 函数。

{% highlight c %}
# if HAVE_CLOCK_SYSCALL
#  ifndef EV_USE_CLOCK_SYSCALL
#   define EV_USE_CLOCK_SYSCALL 1
#   ifndef EV_USE_REALTIME
#    define EV_USE_REALTIME  0
#   endif
#   ifndef EV_USE_MONOTONIC
#    define EV_USE_MONOTONIC 1
#   endif
#  endif
# elif !defined EV_USE_CLOCK_SYSCALL
#  define EV_USE_CLOCK_SYSCALL 0
# endif

# if HAVE_CLOCK_GETTIME
#  ifndef EV_USE_MONOTONIC
#   define EV_USE_MONOTONIC 1
#  endif
#  ifndef EV_USE_REALTIME
#   define EV_USE_REALTIME  0
#  endif
# else
#  ifndef EV_USE_MONOTONIC
#   define EV_USE_MONOTONIC 0
#  endif
#  ifndef EV_USE_REALTIME
#   define EV_USE_REALTIME  0
#  endif
# endif
{% endhighlight %}

通常定义为。

{% highlight text %}
#define HAVE_CLOCK_GETTIME 1
#define EV_USE_REALTIME  0
#define EV_USE_MONOTONIC 1
{% endhighlight %}

在如下的初始化函数中介绍详细的细节。


### 初始化

无论是通过 ```EV_DEFAULT``` 宏还是 ```ev_default_loop()``` 函数进行初始化，实际上功能都相同，也就是都调用了 ```ev_default_loop(0)``` 进行初始化，主要流程为 ```ev_default_loop()->loop_init()``` 。

如下主要介绍 ```loop_init()``` 函数。

{% highlight c %}
#ifndef EV_HAVE_EV_TIME
ev_tstamp
ev_time (void) EV_THROW
{
#if EV_USE_REALTIME
  if (expect_true (have_realtime))
    {
      struct timespec ts;
      clock_gettime (CLOCK_REALTIME, &ts);
      return ts.tv_sec + ts.tv_nsec * 1e-9;
    }
#endif

  struct timeval tv;
  gettimeofday (&tv, 0);
  return tv.tv_sec + tv.tv_usec * 1e-6;
}
#endif

inline_size ev_tstamp
get_clock (void)
{
#if EV_USE_MONOTONIC
  if (expect_true (have_monotonic))
    {
      struct timespec ts;
      clock_gettime (CLOCK_MONOTONIC, &ts);
      return ts.tv_sec + ts.tv_nsec * 1e-9;
    }
#endif

  return ev_time ();
}

void noinline ecb_cold loop_init (EV_P_ unsigned int flags) EV_THROW
{
  if (!backend) {  // 如果backend还没有确定
      origflags = flags;

#if EV_USE_REALTIME
      if (!have_realtime)
        {
          struct timespec ts;

          if (!clock_gettime (CLOCK_REALTIME, &ts))
            have_realtime = 1;
        }
#endif

#if EV_USE_MONOTONIC
      if (!have_monotonic)
        {
          struct timespec ts;

          if (!clock_gettime (CLOCK_MONOTONIC, &ts))
            have_monotonic = 1;
        }
#endif

      /* pid check not overridable via env */
#ifndef _WIN32
      if (flags & EVFLAG_FORKCHECK)
        curpid = getpid ();
#endif

      if (!(flags & EVFLAG_NOENV)
          && !enable_secure ()
          && getenv ("LIBEV_FLAGS"))
        flags = atoi (getenv ("LIBEV_FLAGS"));

      ev_rt_now          = ev_time ();
      mn_now             = get_clock ();
      now_floor          = mn_now;
      rtmn_diff          = ev_rt_now - mn_now;
#if EV_FEATURE_API
      invoke_cb          = ev_invoke_pending;
#endif

      io_blocktime       = 0.;
      timeout_blocktime  = 0.;
      backend            = 0;
      backend_fd         = -1;
      sig_pending        = 0;
#if EV_ASYNC_ENABLE
      async_pending      = 0;
#endif
      pipe_write_skipped = 0;
      pipe_write_wanted  = 0;
      evpipe [0]         = -1;
      evpipe [1]         = -1;
#if EV_USE_INOTIFY
      fs_fd              = flags & EVFLAG_NOINOTIFY ? -1 : -2;
#endif
#if EV_USE_SIGNALFD
      sigfd              = flags & EVFLAG_SIGNALFD  ? -2 : -1;
#endif

      if (!(flags & EVBACKEND_MASK))
        flags |= ev_recommended_backends ();

#if EV_USE_IOCP
      if (!backend && (flags & EVBACKEND_IOCP  )) backend = iocp_init   (EV_A_ flags);
#endif
#if EV_USE_PORT
      if (!backend && (flags & EVBACKEND_PORT  )) backend = port_init   (EV_A_ flags);
#endif
#if EV_USE_KQUEUE
      if (!backend && (flags & EVBACKEND_KQUEUE)) backend = kqueue_init (EV_A_ flags);
#endif
#if EV_USE_EPOLL
      if (!backend && (flags & EVBACKEND_EPOLL )) backend = epoll_init  (EV_A_ flags);
#endif
#if EV_USE_POLL
      if (!backend && (flags & EVBACKEND_POLL  )) backend = poll_init   (EV_A_ flags);
#endif
#if EV_USE_SELECT
      if (!backend && (flags & EVBACKEND_SELECT)) backend = select_init (EV_A_ flags);
#endif

      ev_prepare_init (&pending_w, pendingcb);

#if EV_SIGNAL_ENABLE || EV_ASYNC_ENABLE
      ev_init (&pipe_w, pipecb);
      ev_set_priority (&pipe_w, EV_MAXPRI);
#endif
    }
}
{% endhighlight %}

其中有两个比较重要的时间变量，也就是 ```ev_rt_now``` 和 ```mn_now```，前者表示当前的日历时间，也就是自 1970.01.01 以来的秒数，该值通过 ```gettimeofday()``` 得到。

### 调用流程

在介绍各个 Watcher 的流程之前，首先看下主循环的执行过程。

该函数通常是在各个事件初始化完成之后调用，也就是等待操作系统的事件，然后调用已经注册的回调函数，并一直重复循环执行。

{% highlight c %}
int ev_run (EV_P_ int flags)
{
  ++loop_depth;      // 如果定义了EV_FEATURE_API宏
  loop_done = EVBREAK_CANCEL;
  EV_INVOKE_PENDING; // 在执行前确认所有的事件已经执行

  do {
      ev_verify (EV_A);  // 当EV_VERIFY >= 2时，用于校验当前的结构体是否正常
      if (expect_false (curpid)) /* penalise the forking check even more */
        if (expect_false (getpid () != curpid)) {
            curpid = getpid ();
            postfork = 1;
        }

      /* we might have forked, so queue fork handlers */
      if (expect_false (postfork))
        if (forkcnt) {
            queue_events (EV_A_ (W *)forks, forkcnt, EV_FORK);
            EV_INVOKE_PENDING;
        }

      /* queue prepare watchers (and execute them) */
      if (expect_false (preparecnt)) {
          queue_events (EV_A_ (W *)prepares, preparecnt, EV_PREPARE);
          EV_INVOKE_PENDING;
      }

      if (expect_false (loop_done))
        break;

      /* we might have forked, so reify kernel state if necessary */
      if (expect_false (postfork))
        loop_fork (EV_A);

      /* update fd-related kernel structures */
      fd_reify (EV_A);

      /* calculate blocking time */
      {
        ev_tstamp waittime  = 0.;
        ev_tstamp sleeptime = 0.;

        /* remember old timestamp for io_blocktime calculation */
        ev_tstamp prev_mn_now = mn_now;

        /* 会更新当前时间mn_now和ev_rt_now，如果发现时间被调整，则调用
         * timers_reschedule()函数调整堆loop->timers()中的每个节点。
         */
        time_update (EV_A_ 1e100);

        /* from now on, we want a pipe-wake-up */
        pipe_write_wanted = 1;

        ECB_MEMORY_FENCE; /* make sure pipe_write_wanted is visible before we check for potential skips */

        if (expect_true (!(flags & EVRUN_NOWAIT || idleall || !activecnt || pipe_write_skipped))) {
            waittime = MAX_BLOCKTIME;

            if (timercnt) {    // 如果有定时器存在则重新计算等待时间
                ev_tstamp to = ANHE_at (timers [HEAP0]) - mn_now;
                if (waittime > to) waittime = to;
            }
            if (periodiccnt) { // 如果定义了EV_PERIODIC_ENABLE宏
                ev_tstamp to = ANHE_at (periodics [HEAP0]) - ev_rt_now;
                if (waittime > to) waittime = to;
            }

            /* don't let timeouts decrease the waittime below timeout_blocktime */
            if (expect_false (waittime < timeout_blocktime)) // 默认timeout_blocktime为0
              waittime = timeout_blocktime;

            /* at this point, we NEED to wait, so we have to ensure */
            /* to pass a minimum nonzero value to the backend */
            if (expect_false (waittime < backend_mintime))
              waittime = backend_mintime;  // 不同的后端最小等待时间不同

            /* extra check because io_blocktime is commonly 0 */
            if (expect_false (io_blocktime)) {
                sleeptime = io_blocktime - (mn_now - prev_mn_now);

                if (sleeptime > waittime - backend_mintime)
                  sleeptime = waittime - backend_mintime;

                if (expect_true (sleeptime > 0.)) {
                    ev_sleep (sleeptime);
                    waittime -= sleeptime;
                }
            }
        }

#if EV_FEATURE_API
        ++loop_count;
#endif
        /* 调用IO复用函数，例如epoll_poll()，在此需要保证阻塞时间小于loop->timers，
         * 以及loop->periodics的栈顶元素的触发时间。
         */
        assert ((loop_done = EVBREAK_RECURSE, 1)); /* assert for side effect */
        backend_poll (EV_A_ waittime);
        assert ((loop_done = EVBREAK_CANCEL, 1)); /* assert for side effect */

        pipe_write_wanted = 0; /* just an optimisation, no fence needed */

        ECB_MEMORY_FENCE_ACQUIRE;
        if (pipe_write_skipped) {
            assert (("libev: pipe_w not active, but pipe not written", ev_is_active (&pipe_w)));
            ev_feed_event (EV_A_ &pipe_w, EV_CUSTOM);
        }

        /* update ev_rt_now, do magic */
        time_update (EV_A_ waittime + sleeptime); // 更新时间，防止timejump
      }

      /* 如果栈顶元素的超时时间已经超过了当前时间，则将栈顶元素的监控器添加到
       * loop->pendings中，并调整堆结构，接着判断栈顶元素是否仍超时，一致重复，
       * 直到栈顶元素不再超时。
       */
      timers_reify (EV_A); /* relative timers called last */
      periodics_reify (EV_A); /* absolute timers called first */

      /* queue idle watchers unless other events are pending */
      idle_reify (EV_A);

      /* queue check watchers, to be executed first */
      if (expect_false (checkcnt))
        queue_events (EV_A_ (W *)checks, checkcnt, EV_CHECK);

      /* 按照优先级，顺序遍厉loop->pendings数组，调用其中每个监视器的回调函数 */
      EV_INVOKE_PENDING;
    } while (expect_true (
        activecnt
        && !loop_done
        && !(flags & (EVRUN_ONCE | EVRUN_NOWAIT))
    ));

  if (loop_done == EVBREAK_ONE)
    loop_done = EVBREAK_CANCEL;

#if EV_FEATURE_API
  --loop_depth;
#endif

  return activecnt;
}
{% endhighlight %}



<!--
       - Increment loop depth.
       - Reset the ev_break status.
       - Before the first iteration, call any pending watchers.
       LOOP:
       - If EVFLAG_FORKCHECK was used, check for a fork.
       - If a fork was detected (by any means), queue and call all fork watchers.
       - Queue and call all prepare watchers.
       - If ev_break was called, goto FINISH.
       - If we have been forked, detach and recreate the kernel state
         as to not disturb the other process.
       - Update the kernel state with all outstanding changes.
       - Update the "event loop time" (ev_now ()).
       - Calculate for how long to sleep or block, if at all
         (active idle watchers, EVRUN_NOWAIT or not having
         any active watchers at all will result in not sleeping).
       - Sleep if the I/O and timer collect interval say so.
       - Increment loop iteration counter.
       - Block the process, waiting for any events.
       - Queue all outstanding I/O (fd) events.
       - Update the "event loop time" (ev_now ()), and do time jump adjustments.
       - Queue all expired timers.
       - Queue all expired periodics.
       - Queue all idle watchers with priority higher than that of pending events.
       - Queue all check watchers.
       - Call all queued watchers in reverse order (i.e. check watchers first).
         Signals and child watchers are implemented as I/O watchers, and will
         be handled here by queueing them when their watcher gets executed.
       - If ev_break has been called, or EVRUN_ONCE or EVRUN_NOWAIT
         were used, or there are no active watchers, goto FINISH, otherwise
         continue with step LOOP.
       FINISH:
       - Reset the ev_break status iff it was EVBREAK_ONE.
       - Decrement the loop depth.
       - Return.

    Example: Queue some jobs and then loop until no events are outstanding anymore.

       ... queue jobs here, make sure they register event watchers as long
       ... as they still have work to do (even an idle watcher will do..)
       ev_run (my_loop, 0);
       ... jobs done or somebody called break. yeah!
-->





















<!--
{% highlight text %}
void ev_invoke_pending (struct ev_loop *loop);
  调用所有pending的watchers。

ev_default_loop()/ev_loop_new()
 |-loop_init()
   |-ev_recommended_backends()    如果没有设置backend则会尝试选择
     |-ev_supported_backends()
 |-ev_prepare_init()

ev_run()
 |-backend_poll()
{% endhighlight %}
-->


### IO Watcher

对 IO 事件的监控的函数，会在 loop_init() 中初始化 backend_poll 变量，正是通过该函数监控 io 事件，如下是一个简单的示例。

{% highlight text %}
void cb (struct ev_loop *loop, ev_io *w, int revents)
{
    ev_io_stop (loop, w);
    // .. read from stdin here (or from w->fd) and handle any I/O errors
}
ev_io watcher;
ev_io_init (&watcher, cb, STDIN_FILENO, EV_READ);  // 初始化，第三个是文件描述符，第四个是监听事件
ev_io_start (loop, &watcher);
{% endhighlight %}

其中，ev_io_init() 用来设置结构体的参数，除了初始化通用的变量之外，还包括 io 观察器对应的 fd 和 event 。

#### ev_io_start()

作用是设置 ```ANFD anfds[]```，其中文件描述符为其序号，并将相应的 IO Watcher 插入到对应 fd 的链表中。由于对应 fd 的监控条件已有改动了，同时会在 ```int fdchanges[]``` 中记录下该 fd ，并在后续的步骤中调用系统的接口修改对该 fd 监控条件。

{% highlight c %}
void noinline ev_io_start (EV_P_ ev_io *w) EV_THROW
{
  int fd = w->fd;
  if (expect_false (ev_is_active (w))) // 如果已经启动则直接退出
    return;
  EV_FREQUENT_CHECK;                   // 通过ev_verify()校验数据格式是否正常

  ev_start (EV_A_ (W)w, 1);            // 设置watch->active变量
  array_needsize (ANFD, anfds, anfdmax, fd + 1, array_init_zero);
  wlist_add (&anfds[fd].head, (WL)w);

  // 添加到fdchanges[]数组中
  fd_change (EV_A_ fd, w->events & EV__IOFDSET | EV_ANFD_REIFY);
  w->events &= ~EV__IOFDSET;

  EV_FREQUENT_CHECK;                   // 如上，通过ev_verify()校验数据格式是否正常
}
{% endhighlight %}

![libev io watcher]({{ site.url }}/images/programs/libev_io_watcher_anfds.png "libev io watcher"){: .pull-center }

调用 ev_run() 开始等待事件的触发，该函数中首先会调用 fd_reify()，该函数根据 fdchanges[] 中记录的描述符，将该描述符上的事件添加到 backend 所使用的数据结构中；调用 time_update() 更新当前时间。

接着计算超时时间，并调用 backend_poll() 开始等待事件的发生，如果事件在规定时间内触发的话，则会调用 fd_event() 将触发的监视器记录到 pendings 中；

backend 监听函数 (如 select()、poll()、epoll_wait()等) 返回后，再次调用 time_update() 更新时间，然后调用 ev_invoke_pending() ，依次处理 pendings 中的监视器，调用该监视器的回调函数。



#### fd_reify()

该函数在 ev_run() 的每轮循环中都会调用；会将 fdchanges 中记录的这些新事件一个个的处理，并调用后端 IO 复用的 backend_modify 宏。

<!--
这里需要注意fd_reify()中的思想，anfd[fd] 结构体中，还有一个events事件，它是原先的所有watcher 的事件的 "|" 操作，向系统的epoll 从新添加描述符的操作 是在下次事件迭代开始前进行的，当我们依次扫描fdchangs，找到对应的anfd 结构，如果发现先前的events 与 当前所有的watcher 的"|" 操作结果不等，则表示我们需要调用epoll_ctrl 之类的函数来进行更改，反之不做操作。

实际上 Linux 在分配 fd 时，总是选择系统可用的最小 fd ，所以 anfd 这个数组长度不会太大，而且这个数组会动态分配。

然后启动事件驱动器，最后实际会阻塞到 backend_poll() 中，等待对应的 IO 事件。<br><br>

以 epoll 为例，实际调用的是 ev_epoll.c@epoll_poll() 。当 epoll_wait() 返回一个 fd_event 时 ，就可以直接定位到对应 fd 的 watchers-list ，而这个 watchers-list 的长度与注册的事件相关。<br><br>

fd_event 会有一个导致触发的事件，依次检查对应的 wathers-list ，也即用这个事件依次和各个 watch 注册的 event 做 & 操作，如果不为 0 ，则把对应的 watch 加入到待处理队列 pendings 中。<br><br>

当我们启用 watcher 优先级模式时，pendings 是个 2 维数组，此时仅考虑普通模式。
-->


#### 多路复用

当前支持的多路复用通过如下方式定义，

{% highlight c %}
/* method bits to be ored together */
enum {
  EVBACKEND_SELECT  = 0x00000001U, /* about anywhere */
  EVBACKEND_POLL    = 0x00000002U, /* !win */
  EVBACKEND_EPOLL   = 0x00000004U, /* linux */
  EVBACKEND_KQUEUE  = 0x00000008U, /* bsd */
  EVBACKEND_DEVPOLL = 0x00000010U, /* solaris 8 */ /* NYI */
  EVBACKEND_PORT    = 0x00000020U, /* solaris 10 */
  EVBACKEND_ALL     = 0x0000003FU, /* all known backends */
  EVBACKEND_MASK    = 0x0000FFFFU  /* all future backends */
};
{% endhighlight %}

而在通过 configure 进行编译时，会对宏进行处理，以 epoll 为例，可以查看 ev.c 中的内容；在通过 configure 编译时，如果支持 EPOLL 会在 config.h 中生成 ```HAVE_POLL``` 和 ```HAVE_POLL_H``` 宏定义。

{% highlight c %}
# if HAVE_POLL && HAVE_POLL_H
#  ifndef EV_USE_POLL
#   define EV_USE_POLL EV_FEATURE_BACKENDS
#  endif
# else
#  undef EV_USE_POLL
#  define EV_USE_POLL 0
# endif
{% endhighlight %}

之后调用 ev_recommended_backends() 得到当前系统支持的 backend 类型，比如 select、poll、epoll 等；然后，接下来就是根据系统支持的 backend，按照一定的优先顺序，去初始化 backend 。

<!--
接下来，初始化loop中的ev_prepare监视器pending_w，以及ev_io监视器pipe_w

loop_init返回后，backend已经初始化完成，接着，初始化并启动信号监视器ev_signal childev。暂不深入。

至此，初始化默认loop的工作就完成了。
-->


### Timer Watcher

在 ev_timer_init() 中，分别设置 after 和 repeat 参数，表示 after 秒后执行一次回调函数，之后每隔 repeat 秒执行一次。

{% highlight c %}
//----- 结构体定义，对于时间主要是at+repeat参数
#define EV_WATCHER_TIME(type)      \
  EV_WATCHER (type)                \
  ev_tstamp at;     /* private, 函数初始化对应的after参数 */
typedef struct ev_timer {
  EV_WATCHER_TIME (ev_timer)       // 通用

  ev_tstamp repeat;  /* rw, 函数初始化对应的repeat参数 */
} ev_timer;

//----- 初始化，意味着在after秒后执行，设置为0则会立即执行一次；然后每隔repeat秒执行一次
#define ev_timer_init(ev,cb,after,repeat)        \
    do { ev_init ((ev), (cb)); ev_timer_set ((ev),(after),(repeat)); } while (0)

//----- 如下为一个示例程序
void cb (EV_P_ ev_timer *w, int revents) {
    ev_break(EV_P_ EVBREAK_ONE);         // 实际是设置loop_done的值，也即退出主循环
}
ev_timer watcher;
ev_timer_init (&watcher, cb, 2.5, 1.0);  // 初始化，分别表示多长时间开始执行第一次，后面为时间间隔
ev_timer_start (loop, &watcher);
{% endhighlight %}

在 libev 中的 ev_timer 被放到一个 2-heap 或 4-heap 结构中，这个 heap 结构式存储在数组中，可以参看静态二叉树、最小堆等概念，可以查看 [Binary Heaps](http://www.cs.cmu.edu/~adamchik/15-121/lectures/Binary%20Heaps/heaps.html) 或者 [本地文档](/reference/programs/libev_Binary_Heaps.maff) 。

默认 2-heap ，只有定义了 ```EV_USE_4HEAP``` 宏之后才会使用 4-heap ，后者通常用于数据量大时；对于前者，任一节点，其父节点的位置为 ```floor(k/2)```，任一节点的子节点位置为 ```2*k``` 和 ```2*k+1``` 。

它是一个最小堆，权值为 "即将触发的时刻"，所以其根节点总是最近要触发的 timer；对此堆有两个基本操作，upheap() 和 downheap()。

#### 最小栈

当对某一节点执行 upheap() 时，就是与其父节点进行比较，如果其值比父节点小，则交换，然后在对这个父节点重复 upheap() ，直到顶层。

downheap() 操作会与子节点比较，如果子节点中有小于当前节点的权，则选择最小的节点进行交换，并一直重复。

![libev timer watcher]({{ site.url }}/images/programs/libev_timer_watcher.png "libev timer watcher"){: .pull-center }

在 ```ev_timer_start()``` 函数中，会将定时器监控器注册到事件驱动器上。

#### 函数执行流程

如上所述。

{% highlight c %}
void noinline ev_timer_start (EV_P_ ev_timer *w) EV_THROW
{
  if (expect_false (ev_is_active (w)))
    return;
  ev_at (w) += mn_now;

  EV_FREQUENT_CHECK;

  ++timercnt;
  ev_start (EV_A_ (W)w, timercnt + HEAP0 - 1);
  array_needsize (ANHE, timers, timermax, ev_active (w) + 1, EMPTY2);
  ANHE_w (timers [ev_active (w)]) = (WT)w;
  ANHE_at_cache (timers [ev_active (w)]);
  upheap (timers, ev_active (w));

  EV_FREQUENT_CHECK;
}
{% endhighlight %}

<!--
其首先 ev_at (w) += mn_now; 得到未来的时间，这样放到时间管理的堆“timers”中作为权重。然后通过之前说过的“ev_start”修改驱动器loop的状态。这里我们又看到了动态大小的数组了。Libev的堆的内存管理也是通过这样的关系的。具体这里堆的实现，感兴趣的可以仔细看下实现。这里的操作就是将这个时间权重放到堆中合适的位置。这里堆单元的结构为：

 其实质就是一个时刻at上挂一个放定时器watcher的list。当超时时会依次执行这些定时器watcher上的触发回调函数。
1.4定时器监控器的触发

最后看下在一个事件驱动器循环中是如何处理定时器监控器的。这里我们依然抛开其他的部分，只找定时器相关的看。在“/ calculate blocking time /”块里面，我们看到计算blocking time的时候会先：

点击(此处)折叠或打开

    if (timercnt) {
        ev_tstamp to = ANHE_at (timers [HEAP0]) - mn_now;
        if (waittime > to) waittime = to;
    }


如果有定时器，那么就从定时器堆（一个最小堆）timers中取得堆顶上最小的一个时间。这样就保证了在这个时间前可以从backend_poll中出来。出来后执行timers_reify处理将pengding的定时器。

在timers_reify中依次取最小堆的堆顶，如果其上的ANHE.at小于当前时间，表示该定时器watcher超时了，那么将其压入一个数组中，由于在实际执行pendings二维数组上对应优先级上的watcher是从尾往头方向的，因此这里先用一个数组依时间先后次存下到一个中间数组loop->rfeeds中。然后将其逆序调用ev_invoke_pending插入到pendings二维数组中。这样在执行pending事件的触发动作的时候就可以保证，时间靠前的定时器优先执行。函数 feed_reverse和 feed_reverse_done就是将超时的定时器加入到loop->rfeeds暂存数组以及将暂存数组中的pending的watcher插入到pengdings数组的操作。把pending的watcher加入到pendings数组，后续的操作就和之前的一样了。回依次执行相应的回调函数。

这个过程中还判断定时器的 w->repeat 的值，如果不为0，那么会重置该定时器的时间，并将其压入堆中正确的位置，这样在指定的时间过后又会被执行。如果其为0，那么调用ev_timer_stop关闭该定时器。 其首先通过clear_pending置pendings数组中记录的该watcher上的回调函数为一个不执行任何动作的哑动作。

总结一下定时器就是在backend_poll之前通过定时器堆顶的超时时间，保证blocking的时间不超过最近的定时器时间，在backend_poll返回后，从定时器堆中取得超时的watcher放入到pendings二维数组中，从而在后续处理中可以执行其上注册的触发动作。然后从定时器管理堆上删除该定时器。最后调用和ev_start呼应的ev_stop修改驱动器loop的状态，即loop->activecnt减少一。并将该watcher的active置零。

对于周期性的事件监控器是同样的处理过程。只是将timers_reify换成了periodics_reify。其内部会对周期性事件监控器派生类的做类似定时器里面是否repeat的判断操作。判断是否重新调整时间，或者是否重复等逻辑，这些看下代码比较容易理解，这里不再赘述。·
-->

### Periodic Wather

这是绝对时间定时器，不同于 ```ev_timer```，它是基于日历时间的；例如，指定一个 ```ev_periodic``` 在 10 秒之后触发 ```(ev_now()+10)```，然后在 10 秒内将系统时间调整为去年，则该定时器会在一年后才触发超时事件，而 ```ev_timer``` 依然会在 10 秒之后触发。

首先看下 libev 中定义的结构体。

{% highlight c %}
#define EV_WATCHER(type)              \
  int active; /* private */           \
  int pending; /* private */          \
  EV_DECL_PRIORITY /* private */      \
  EV_COMMON /* rw */                  \
  EV_CB_DECLARE (type) /* private */

#define EV_WATCHER_TIME(type)         \
  EV_WATCHER (type)                   \
  ev_tstamp at;     /* private */

typedef struct ev_periodic
{
  EV_WATCHER_TIME (ev_periodic)

  ev_tstamp offset; /* rw */
  ev_tstamp interval; /* rw */
  ev_tstamp (*reschedule_cb)(struct ev_periodic *w, ev_tstamp now) EV_THROW; /* rw */
} ev_periodic;

// 上述结构体实际上等价于如下
typedef struct ev_periodic
{
    int active;
    int pending;
    int priority;
    void *data;
    void (*cb)(struct ev_loop *loop, struct ev_periodic *w, int revents);
    ev_tstamp at;

    ev_tstamp offset; /* rw */
    ev_tstamp interval; /* rw */
    ev_tstamp (*reschedule_cb)(struct ev_periodic *w, ev_tstamp now) EV_THROW; /* rw */
} ev_periodic;
{% endhighlight %}

如上结构体，其中前六个成员与 ev_timer 一样，而且offset、interval和reschedule_cb都是用来设置触发时间的，这个会在下面说明。

#### 初始化

与其它 Watcher 相似，初始化可通过 ev_init()+ev_periodic_set() 或直接通过 ev_periodic_init() 初始化，可以查看如下内容。

{% highlight c %}
#define ev_init(ev,cb_) do {                      \
  ((ev_watcher *)(void *)(ev))->active  =         \
  ((ev_watcher *)(void *)(ev))->pending = 0;      \
  ev_set_priority ((ev), 0);                      \
  ev_set_cb ((ev), cb_);                          \
} while (0)

#define ev_periodic_set(ev,ofs_,ival_,rcb_)  do { \
  (ev)->offset = (ofs_);                          \
  (ev)->interval = (ival_);                       \
  (ev)->reschedule_cb = (rcb_);                   \
} while (0)

#define ev_periodic_init(ev,cb,ofs,ival,rcb) do { \
  ev_init ((ev), (cb));                           \
  ev_periodic_set ((ev),(ofs),(ival),(rcb));      \
} while (0)
{% endhighlight %}

#### 启动定时器

其中启动函数如下。

{% highlight c %}
void noinline ev_periodic_start (EV_P_ ev_periodic *w) EV_THROW
{
  if (expect_false (ev_is_active (w)))
    return;

  if (w->reschedule_cb)
    ev_at (w) = w->reschedule_cb (w, ev_rt_now);
  else if (w->interval)
    {
      assert (("libev: ev_periodic_start called with negative interval value", w->interval >= 0.));
      periodic_recalc (EV_A_ w);
    }
  else
    ev_at (w) = w->offset;

  EV_FREQUENT_CHECK;

  ++periodiccnt;
  ev_start (EV_A_ (W)w, periodiccnt + HEAP0 - 1);
  array_needsize (ANHE, periodics, periodicmax, ev_active (w) + 1, EMPTY2);
  ANHE_w (periodics [ev_active (w)]) = (WT)w;
  ANHE_at_cache (periodics [ev_active (w)]);
  upheap (periodics, ev_active (w));

  EV_FREQUENT_CHECK;
}
{% endhighlight %}

共有三种设置超时时间 at 的方法，也就是：

<ol><li>

如果 reschedule_cb() 不为空，则忽略 interval 和 offset，而使用该函数设置超时时间 at，该函数以 ev_rt_now 为参数，设置下次超时事件触发的时间，示例程序如下。

{% highlight text %}
static ev_tstamp my_rescheduler (ev_periodic *w, ev_tstamp now)
{
    return now + 60.;
}
{% endhighlight %}

这也就是将 at 设置为 1 分钟之后的时间点开始。</li><br><li>

当 reschedule_cb() 为空且 interval>0 时，调用 periodic_recalc() 函数设置 at，也就是将 at 设置为下一个的 offset+N*interval 时间点，其中的 offset 一般处于 [0, interval] 范围内。<br>

比如置 offset 为 0，interval 为 3600，意味着当系统时间是完整的 1 小时的时候，也就是系统时间可以被 3600 整除的时候，比如 8:00、9:00 等，就会触发超时事件。</li><br><li>

如果 reschedule_cb 为空且 interval 为 0，则直接将 at 置为 offset，此时不会重复触发，触发一次之后就会停止；而且该监视器也会无视时间调整，比如置 at 为 20110101000000，则只要系统日历时间超过了改时间，就会触发超时事件。
</li></ol>

设置好 at 后，就将该监视器加入到堆 periodics 中，这与 ev_timer 的代码是一样的，不再赘述。

#### 计算触发点时间

如下的 periodic_recalc() 函数会重新计算下一个触发时间点。

{% highlight c %}
static void noinline periodic_recalc (EV_P_ ev_periodic *w)
{
  ev_tstamp interval = w->interval > MIN_INTERVAL ? w->interval : MIN_INTERVAL;
  ev_tstamp at = w->offset + interval * ev_floor ((ev_rt_now - w->offset) / interval);

  /* the above almost always errs on the low side */
  while (at <= ev_rt_now) {
      ev_tstamp nat = at + w->interval;

      /* when resolution fails us, we use ev_rt_now */
      if (expect_false (nat == at)) {
          at = ev_rt_now;
          break;
      }

      at = nat;
  }

  ev_at (w) = at;
}
{% endhighlight %}

该函数的作用就是将 at 置为下一个的 offset+N*interval 时间点，其中 ```ev_floor(x)``` 返回小于 x，且最接近 x 的整数。

<!--
举个例子可能会容易明白该代码：interval为10分钟（600），offset为2分钟（120），表示将at置为下一个分钟数为2的时间点。

假设当前为8:01:23，则最终会使得at为8:02:00。计算过程是 ：interval * ev_floor ((ev_rt_now - w->offset) / interval)就表示7:50:00，然后再加上offset就是7:52:00，进入循环，最终调整得at=8:02:00。

假设当前为8:03:56，则最终会使得at为8:12:00。计算过程是：interval * ev_floor ((ev_rt_now -w->offset) / interval)就表示8:00:00，然后再加上offset就是8:02:00，进入循环，最终调整得at=8:12:00。
-->

#### 超时时间调整

通过 periodics_reschedule() 函数，用于重新调整超时时间。

{% highlight c %}
static void noinline ecb_cold periodics_reschedule (EV_P)
{
  int i;

  /* adjust periodics after time jump */
  for (i = HEAP0; i < periodiccnt + HEAP0; ++i)
    {
      ev_periodic *w = (ev_periodic *)ANHE_w (periodics [i]);

      if (w->reschedule_cb)
        ev_at (w) = w->reschedule_cb (w, ev_rt_now);
      else if (w->interval)
        periodic_recalc (EV_A_ w);

      ANHE_at_cache (periodics [i]);
    }

  reheap (periodics, periodiccnt);
}
{% endhighlight %}

在 time_update() 函数中，如果发现日历时间被调整了，则会通过调用 periodics_reschedule() 调整 ev_periodic 的超时时间点 at；调整的方法与启动函数中一样，要么使用 reschedule_cb() 调整，要么调用 periodic_recalc() 重新计算 at。

最后，将 periodics 堆中所有元素都调整完毕后，调用 reheap() 使 periodics 恢复堆结构。

<!--
http://blog.csdn.net/gqtcgq/article/details/49531625
####

将激活的超时事件排队periodics_reify

主要流程跟timers_reify一样，只不过在重新计算下次触发时间点at的时候，计算方法跟ev_periodic_start中的一样。
-->

### Signal Watcher

在收到 SIGINT 时做些清理，直接退出。

{% highlight text %}
static void sigint_cb (struct ev_loop *loop, ev_signal *w, int revents)
{
    ev_break (EV_A_ EVBREAK_ALL);
}
ev_signal signal_watcher;
ev_signal_init (&signal_watcher, sigint_cb, SIGINT);
ev_signal_start (loop, &signal_watcher);
{% endhighlight %}

### Child Watcher

fork 一个新进程，给它安装一个child处理器等待进程结束。

{% highlight text %}
ev_child cw;
static void child_cb (EV_P_ ev_child *w, int revents)
{
    ev_child_stop (EV_A_ w);
    printf ("process %d exited with status %x\n", w->rpid, w->rstatus);
}
pid_t pid = fork ();
if (pid < 0) {            // error
    perror("fork()");
    exit(EXIT_FAILURE);
} else if (pid == 0) {    // child
    // the forked child executes here
    sleep(1);
    exit (EXIT_SUCCESS);
} else {                  // parent
    ev_child_init (&cw, child_cb, pid, 0);
    ev_child_start (EV_DEFAULT_ &cw);
}
{% endhighlight %}

### Filestat Watcher

监控 Makefile 是否有变化，可以通过修改文件触发事件。

{% highlight text %}
static void filestat_cb (struct ev_loop *loop, ev_stat *w, int revents)
{
    // "Makefile" changed in some way
    if (w->attr.st_nlink) {
        printf ("Makefile current size  %ld\n", (long)w->attr.st_size);
        printf ("Makefile current atime %ld\n", (long)w->attr.st_mtime);
        printf ("Makefile current mtime %ld\n", (long)w->attr.st_mtime);
    } else { /* you shalt not abuse printf for puts */
        puts ("wow, Makefile is not there, expect problems. "
              "if this is windows, they already arrived\n");
    }
}
ev_stat makefile;
ev_stat_init (&makefile, filestat_cb, "Makefile", 0.);
ev_stat_start (loop, &makefile);
{% endhighlight %}

## 代码优化

libev 可以通过很多宏进行调优，默认会通过 EV_FEATURES 宏定义一些特性，定义如下。

{% highlight c %}
#ifndef EV_FEATURES
# if defined __OPTIMIZE_SIZE__
#  define EV_FEATURES 0x7c  /* 0111 1100 */
# else
#  define EV_FEATURES 0x7f  /* 0111 1111 */
# endif
#endif

#define EV_FEATURE_CODE     ((EV_FEATURES) &  1) /* 0000 0001 */
#define EV_FEATURE_DATA     ((EV_FEATURES) &  2) /* 0000 0010 */
#define EV_FEATURE_CONFIG   ((EV_FEATURES) &  4) /* 0000 0100 */
#define EV_FEATURE_API      ((EV_FEATURES) &  8) /* 0000 1000 */
#define EV_FEATURE_WATCHERS ((EV_FEATURES) & 16) /* 0001 0000 */
#define EV_FEATURE_BACKENDS ((EV_FEATURES) & 32) /* 0010 0000 */
#define EV_FEATURE_OS       ((EV_FEATURES) & 64) /* 0100 0000 */
{% endhighlight %}


## 参考

源码可以从 [freenode - libev](http://freecode.com/projects/libev) 上下载，不过最近的更新是 2011 年，也可以从 [github](https://github.com/enki/libev) 上下载，或者下载 [本地保存版本 libev-4.22](/reference/linux/libev-4.22.tar.bz2)；帮助文档可以参考 [本地文档](/reference/linux/libev.html) 。

对于 python ，提供了相关的扩展 [Python libev interface - pyev](http://packages.python.org/pyev/) 。

<!--
libev and libevent对比
https://blog.gevent.org/2011/04/28/libev-and-libevent/
-->


{% highlight text %}
{% endhighlight %}

---
title: Collectd 源码解析
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: collectd,monitor,linux
description: 介绍下 Collectd 中源码的实现。
---

接下来介绍下 Collectd 中源码的实现。

<!-- more -->


## 常用概念

在介绍源码的实现前，首先看下一些常见的概念。

插件采集的数据保存在 value_list_t 中，这个对应了 types.db 中的一行记录，types.db 中一行可能会包含了多个采集结构，例如 load 指标，那么与之对应 value_list_t 也可能会包含了多个采集值。

### 标示符 (Identifier)

value_list_t 中还包含了用于构建标示符 (Identifier) 的值，包括了: host、plugin、plugin instance (optional)、type、type instance (optional) 五部分，其中两个是可选的，其含义如下：

* host 主机名称，可以再插件中设置，否则直接获取配置文件中的主机名；
* plugin 一般是插件名称；
* plugin instance 可选，例如 CPU 在多核的场景；
* type 很多还是和插件相同，感觉有些冲突；
* type instance 可选，监控的不同类型，例如 CPU 的指标有 idle、usage 等。

其中每项命名的最大值为 64，建议不要使用 ```'/'``` 和 ```'-'```，格式化时通过 ```format_name()``` 实现，串行化的格式为：```host "/" plugin ["-" plugin instance] "/" type ["-" type instance]``` 。

如下，是几个常见的示例：

{% highlight text %}
localhost/cpu-0/cpu-idle
localhost/memory/memory-used
wanda/disk-hdc1/disk_octets
leeloo/load/load
{% endhighlight %}

常见的在 global cache 中会使用，用于保存最近保存的一次数据，与 notification 相关的参数。

个人建议使用 Plugin + Plugin Instance + types.db 这种方式定义 Identifier 。


### 采集值类型

Collectd 中总共有四种类型，简单介绍如下：

* GAUSE: 直接使用采样值，通常为温度、内存使用率等参数值；
* DERIVE: 计算采样的速率，计算公式为 ```rate=(value_new-value_old)/(time_new-time_old)``` ，需要注意，如果值非递增，那么可能产生负值；
* COUNTER: 与 DERIVE 相同，只是当 ```value_new<value_old``` 时则认为是 "wrapped around" <!--，此时会按照 ```rate=(2**width-value_old+value_new)/(time_new-time_old)width=value_old< 2**32 ? 32 : 64```--> 。
* ABSOULUTE: 与 GAUSE 比较相似，只是该值是无符号整数，且会除以当前时间与上次采样时间的差值；

需要注意的是，COUNTER 值采集的三种特殊情况：1) 计数器达到最大值，导致回环；2) 监控服务维护(重启或者手动计数器清零)；3) 监控 agent 重启，导致上次计数丢失。这三种情况可能会导致采集指标异常，通常只有一个采集点，一般可以过滤或者忽略掉。

<!-- 详见: https://collectd.org/wiki/index.php/Data_source -->

## 插件实现

首先，简单说下插件是如何加载的。

有个配置项 ```AutoLoadPlugin false``` 用于控制是否会自动加载插件，默认需要先通过 LoadPlugin 指令加载插件，然后再在 ```<Plugin XXX>``` 中进行配置，否则会报错；当然也可以将该参数修改为 true ，然后在遇到 ```<Plugin XXX>``` 时自动加载。

源码中有一个示例插件，可查看 ```contrib/examples``` 目录下的文件；当然，也可以参考 [官方文档](https://collectd.org/wiki/index.php/Plugin_architecture) 。

插件的实现实际上主要是通过将回调函数注册到链表上。

{% highlight text %}
int plugin_register_init()<<<1>>>
注册初始化函数，正式创建读写线程之前执行。
使用链表: list_init

int plugin_register_config()<<<2>>>
简单的配置处理，只含有配置项以及回调函数。
使用链表: first_callback

int plugin_register_complex_config()<<<2>>>
通过一个注册的回调函数进行配置解析，在链表末尾写入；该函数会对配置项进行检测，包括了一些复杂的逻辑处理。
使用链表: complex_callback_head

int plugin_register_flush()<<<3>>>
有部分插件会缓存数据，例如network、rrdtool等，通过函数注册到链表，在执行 flush 命令时会调用链表上的各个函数。
使用链表: list_flush，通过plugin_flush()函数调用

int plugin_register_read()<<<3>>>
int plugin_register_complex_read()
两个函数的区别在于回调函数中是否需要传入参数，详见plugin_read_thread()中的调用。
使用链表：read_list

int plugin_register_write()<<<3>>>
写入插件的回调函数注册。
使用链表：list_write

int plugin_register_missing()<<<3>>>
会在每次的时间间隔检查是否有数据丢失，当发现有数据丢失时会调用注册的函数。
使用链表：list_missing

int plugin_register_shutdown()<<<4>>>
通过plugin_shutdown_all()函数调用。
使用链表：list_shutdown
{% endhighlight %}

<!--
int plugin_register_data_set()
int plugin_register_log()
int plugin_register_notification()
https://collectd.org/wiki/index.php/Plugin_architecture
-->


## 源码详解

介绍下调用流程。

### 内存管理

这里以 load 插件为例。

1. 通过 ```plugin_register_read()``` 函数注册到链表中，调用回调函数采集指标；
2. 回调函数，load 通过 ```VALUE_LIST_INIT``` 宏初始化，也就是初始化一个本地的临时变量，然后通过 ```plugin_dispatch_values()``` 函数发送给 collectd 核心；
3. 在 ```plugin_write_enqueue()``` 函数中主要的内存分配函数，包括了 ```malloc(sizeof(write_queue_t)) + malloc(sizeof(value_list_t)) + calloc(len,sizeof(values)) + calloc(1,sizeof(meta_data_t))```，如果中间有内存分配失败则释放；
4. 而写线程插件会阻塞在 ```plugin_write_dequeue()``` 中，该函数会释放 ```sfree(write_queue_t)``` ，然后返回 ```value_list_t``` 指针；
5. 线程的主循环 ```plugin_write_thread()``` 函数中，在执行完分发后会调用 ```plugin_value_list_free()``` 释放之前申请的主要内存。

calloc() 会将申请的缓存初始化为 0 ，而 malloc() 不会初始化内存。

### 调用流程

首先看下整个软件调用流程。

{% highlight text %}
main()
 | <<<<<<<<<========= 读取配置文件
 |-plugin_init_ctx()
 |-cf_read()                                  ← 读取配置文件
 | |-cf_read_generic()                        ← 判断文件是否存在
 | | |-cf_read_file()                         ← 读取配置文件内容
 | | |-oconfig_parse_file()                   ← 解析配置项
 | |   |-fopen()
 | |   |-oconfig_parse_fh()
 | |   | |-yyparse()                          ← 使用LEX+YACC进行词法+语法解析
 | |   |-fclose()
 | |
 | |-dispatch_value()                         ← 没有子配置项，一般为全局配置
 | | |-dispatch_global_option()               ← 全局配置，也就是cf_global_options变量中的定义
 | | | |-global_option_set()                  ← 设置全局变量
 | | |-dispatch_value_typesdb()               ← 对于cf_value_map中的定义，例如Typesdb，解析types.db
 | |   |-read_types_list()                    ← 读取一个配置文件，可以通过TypesDB指定多个配置文件
 | |     |-fopen()
 | |     |-parse_file()
 | |     | |-fgets()
 | |     | |-parse_line()
 | |     |   |-plugin_register_data_set()     ← 注册data_set_t类型，通过avl保存在data_sets全局变量中
 | |     |-fclose()
 | |
 | |-dispatch_block()                         ← 如果是配置块
 |   |-dispatch_loadplugin()                  ← LoadPlugin，会调用plugin_load()
 |   | |-plugin_load()
 |   |   |-plugin_load_file()
 |   |     |-lt_dlsym()                       ← 调用各个插件的module_register()函数
 |   |       |-plugin_register_complex_read() ← 会生成read_func_t对象
 |   |         |-plugin_insert_read()         ← 写入到list以及heap
 |   |
 |   |-dispatch_block_plugin()                ← Plugin，会调用plugin_load()
 |   | |-plugin_load()                        ← 需要配置AutoLoadPlugin(true)参数
 |   | | ### 调用complex_callback_head链表
 |   | |-dispatch_value_plugin()
 |   |   |-cf_dispatch()
 |   |     |-cf_search()                      ← 查找first_callback链表中的回调函数
 |   |
 |   |-fc_configure()                         ← Chain，模块
 |     |-fc_init_once()
 |     |-fc_config_add_chain()
 |
 | <<<<<<<<<========= 系统初始化
 |-change_basedir()                           ← 如果设置了basedir，则尝试切换
 | |-chdir()                                  ← 切换到指定目录下
 |-init_global_variables()                    ← 初始化全局变量，如interval_g、timeout_g、hostname
 | |-cf_get_default_interval()                ← 设置全局的时间间隔
 | |-init_hostname()                          ← 获取主机名
 |   |-global_option_get()                    ← 尝试从配置文件中通过Hostname选项获取主机名
 |   |-gethostname()                          ← 没有配置则尝试通过该系统调用获取
 |   |-global_option_get()                    ← 通过FQDNLookup确认是否需要查找主机名
 |   |-getaddrinfo()                          ← 如果上述配置为true
 |-sigaction()                                ← 注册信号处理函数，包括了SIGINT、SIGTERM、SIGUSR1
 |
 | <<<<<<<<<========= 创建工作线程
 |-do_init()
 | |-plugin_init_all()                        ← 会设置缓存队列的大小
 |   |-uc_init()                              ← 初始化全局变量cache_tree(平衡二叉树)
 |   | ###BEGIN读取所需要的配置参数
 |   |-plugin_register_read()                 ← 如果已经配置CollectInternalStats，则注册一个读取插件
 |   |-fc_chain_get_by_name()                 ← 读取PreCacheChain、PostCacheChain配置参数
 |   |
 |   |-cf_callback()                          ← <<<1>>>调用list_init链表中回调函数，检测插件是否正常
 |   |
 |   |-start_write_threads()                  ← 开启写线程
 |   | |-plugin_write_thread()                ← 启动写线程，真正的线程执行函数
 |   | | | ###BEGIN###while循环调用，判断write_loop是否可用
 |   | | |-plugin_write_dequeue()             ← 从write_queue链表的首部获取一个对象，队列递减
 |   | | | |-pthread_cond_wait()              ← 等待write_cond条件变量，使用write_lock锁
 |   | | |-plugin_dispatch_values_internal()  ← 入参是value_list_t类型
 |   | | | |-c_avl_get()                      ← 从data_sets中获取对应的数据类型，也就是types.db中
 |   | | | |-escape_slashes()                 ← 处理host、plugin、type中可能的转移字符
 |   | | | |-fc_process_chain()               ← 如果pre_cache_chain不为空，则调用该函数
 |   | | | |-uc_update()                      ← 更新缓存cache_tree，入参为data_set_t和value_list_t
 |   | | | | |-FORMAT_VL()                    ← 实际上是调用format_name()将vl中的值生成标示符
 |   | | | | |-c_avl_get()                    ← 利用上述标示符获取cache_entry_t，在此会缓存最近的一次采集数据
 |   | | | | |... ...                         ← 会根据不同的类型进行处理，例如DS_TYPE_GAUGE
 |   | | | | |-uc_check_range()               ← 检查是否在指定的范围内
 |   | | | |-fc_process_chain()  ##OR1##      ← 如果有post_cache_chain则调用
 |   | | | |-fc_default_action() ##OR1##      ← 调用写入插件写入
 |   | | |   |-fc_bit_write_invoke()
 |   | | |     |-plugin_write()               ← 通过list_write链表调用各个组件写入
 |   | | |       |-cf_callback()
 |   | | | ###END###while
 |   | | |-plugin_value_list_free()           ← TODODO:是否使用缓存池
 |   | |-set_thread_name()                    ← 设置线程名称writerN
 |   |
 |   |-start_read_threads()                   ← 创建多个读线程，需要保证read_heap!=NULL
 |     |-plugin_read_thread()                 ← 线程实际入口，在一个死循环中执行，详细见后面的解析
 |     | | ###BEGIN###while循环调用，判断read_loop是否可用
 |     | |-pthread_cond_wait()                ← 等待read_cond条件变量，使用read_lock锁
 |     | |-c_heap_get_root()
 |     | | ###END###
 |     |-set_thread_name()                    ← 设置线程名称writerN
 |
 |-test_readall()                             ← 如果使用参数-T，则调用插件读取一行
 | |-plugin_read_all_once()                   ← 调用各个插件的读取函数，测试是否有误
 |   |-c_heap_get_root()
 |   |-rf_callback()                          ← 调用各个插件的读取函数
 |
 |-do_loop()                                  ← 在此包括了时间间隔
 | |-cf_get_default_interval()                ← 获取默认的时间间隔
 | |-cdtime()
 | | ###BEGIN###while循环调用，判断loop是否可用
 | |-plugin_read_all()
 |   |-uc_check_timeout()                     ← 主线程检查是否有插件超时
 |     |-plugin_dispatch_missing()            ← 如果有超时，则调用注册的回调函数
 |
 |-do_shutdown()                              ← 通过INT、TERM信号用于关闭
   |-plugin_shutdown_all()                    ← 调用链表上回调函数
     |-destroy_all_callbacks()

----- 注册简单读取插件
plugin_register_read()                        ← 新建read_func_t对象
 |-plugin_insert_read()
   |-llist_create()                           ← 如果第一次调用，则生成read_list
   |-c_heap_create()                          ← 如果第一次调用，则生成read_heap
   |-llist_search()                           ← 是否重复注册，直接返回
   |-c_heap_insert()                          ← 写入read_heap，通过rf_next_read排序，也即下次要读取时间

----- 各个插件在采集完数据之后用于保存数据
plugin_dispatch_values()
 |-check_drop_value()                         ← 检查是否会丢弃数据
 | |-get_drop_probability()                   ← 用于计算是否丢弃
 |-plugin_write_enqueue()                     ← 直接写入到write_queue链表的末尾
   |-plugin_value_list_clone()                ← 复制一份，会自动填充主机名、采集时间、采样频率
   | |-malloc()
   | |-meta_data_clone()
   |-pthread_cond_signal()                    ← 添加到write_queue链表中，并发送write_cond

----- 注册简单写回调函数，保存到list_write链表中
plugin_register_write()
 |-create_register_callback()
   |-calloc()                                 ← 分配插件需要空间
   |-register_callback()                      ← 添加到list_write链表中

----- 注册到链表中，部分需要插件需要执行刷新，如rrdtool、network会注册该函数
plugin_register_flush()
 |-create_register_callback()                 ← 添加到list_flush链表中
 |-plugin_register_complex_read()             ← 如果刷新时间间隔不为0，则调用该插件注册
                                              ← 回调函数plugin_flush_timeout_callback()
{% endhighlight %}

<!--
cache_tree中的值什么时候开始刷新？保存了什么样的数据？
线程切换时上下文的调用？
-->

### 读线程

在通过 ```plugin_register_read()``` 函数进行注册时，会生成 read_list 和 read_heap 两个全局变量，其中 read_list 只是用来维护注册的读插件，实际用途不大；而 read_heap 类似于堆排序，用于获取最近需要进行读取的插件，也就是说，下个时间点调用那个函数，通过 heap 进行排序。

如果调用插件读取失败，则会采用 double 的退避措施，也就是将下次采集时间乘以 2，直到 max_read_interval 值；读取成功时则会直接恢复原有的采集时间。

{% highlight c %}
static void *plugin_read_thread(void __attribute__((unused)) * args) {
  while (read_loop != 0) {
    // 从read_heap中读取排序在前的对象，一般只是由于多线程之间竞争导致阻塞
    // 不过，还有可能会存在线程数大于插件数的情况，此时就会进入条件等待
    pthread_mutex_lock(&read_lock);
    rf = c_heap_get_root(read_heap);
    if (rf == NULL) {
      pthread_cond_wait(&read_cond, &read_lock);
      pthread_mutex_unlock(&read_lock);
      continue;
    }
    pthread_mutex_unlock(&read_lock);

    // 也就是监控的时间间隔，正常会在插件加载的时候已经配置
    // 默认是采用全局的配置，不过也可以在插件配置中设置自己的参数
    if (rf->rf_interval == 0) {
      rf->rf_interval = plugin_get_interval();
      rf->rf_effective_interval = rf->rf_interval;
      rf->rf_next_read = cdtime();
    }

    // 正常来说我们可以在此执行nsleep()做等待，但是在关闭读线程时，仍使用read_cond
    pthread_mutex_lock(&read_lock);
    // 对于Linux来说，不会有惊群现象，但是对于NetBSD而且CPU>1时会存在问题
    rc = 0;
    while ((read_loop != 0) && (cdtime() < rf->rf_next_read) && rc == 0) {
      rc = pthread_cond_timedwait(&read_cond, &read_lock,
                                  &CDTIME_T_TO_TIMESPEC(rf->rf_next_read));
    }
    rf_type = rf->rf_type;          // 需要持有read_lock读取
    pthread_mutex_unlock(&read_lock);

    // 如上所述，此时可能是需要退出的，在此检查下
    if (read_loop == 0) {
      c_heap_insert(read_heap, rf); // 为了可以正常free，需要将rf重新插入到read_heap中
      break;
    }

    // 在plugin_unregister_read()函数中，已经将该插件删除，此时只需要删除即可
    if (rf_type == RF_REMOVE) {
      DEBUG("plugin_read_thread: Destroying the `%s' callback.", rf->rf_name);
      sfree(rf->rf_name);
      destroy_callback((callback_func_t *)rf);
      rf = NULL;
      continue;
    }

    // OK，开始正式调用插件读取数据了
    DEBUG("plugin_read_thread: Handling `%s'.", rf->rf_name);
    start = cdtime();
    old_ctx = plugin_set_ctx(rf->rf_ctx);
    if (rf_type == RF_SIMPLE) {
      int (*callback)(void);
      callback = rf->rf_callback;
      status = (*callback)();
    } else {
      plugin_read_cb callback;
      assert(rf_type == RF_COMPLEX);
      callback = rf->rf_callback;
      status = (*callback)(&rf->rf_udata);
    }
    plugin_set_ctx(old_ctx);


    // 如果失败则会将下次采集的时间间隔double，当然是有上限的；成功时则会恢复原有的采集时间间隔
    if (status != 0) {
      rf->rf_effective_interval *= 2;
      if (rf->rf_effective_interval > max_read_interval)
        rf->rf_effective_interval = max_read_interval;
    } else {
      rf->rf_effective_interval = rf->rf_interval;
    }
    now = cdtime();
    elapsed = (now - start);  // 计算本次花费的时间
    if (elapsed > rf->rf_effective_interval)
      WARNING(... ...);

    // 计算下次需要调用插件的时间
    rf->rf_next_read += rf->rf_effective_interval;

    // 如果采集时间超过了时间间隔，则立即重新采集
    if (rf->rf_next_read < now) {
      rf->rf_next_read = now;
    }

    // 重新写入到read_heap中，到此插件调用结束
    c_heap_insert(read_heap, rf);
  } /* while (read_loop) */

  pthread_exit(NULL);
  return ((void *)0);
} /* void *plugin_read_thread */
{% endhighlight %}

从上述的函数调用可知，collectd 框架不会负责数据采集写入，需要由各个插件负责。

### 插件采集数据

实际上，在 contrib/examples 目录下有个插件的示例程序；在此选一个比较简单的插件 load，一般最终会通过 ```plugin_dispatch_values()``` 函数提交。该函数主要是将数据添加到 write_queue_head 列表中，并发送 write_cond。

## FAQ

1\. meta data 的作用是？

meta data (meta_data_t) 用于将一些数据附加到已经存在的结构体上，如 values_list_t，对应的是 KV 结构，其中一个使用场景是 network 插件，用于标示从哪个插件采集来的，防止出现循环，也用于该插件的 Forward 选项。


<!--
### 线程上下文

TODO:保存的什么信息？？？？

plugin_init_ctx()
 |-pthread_key_create()
 |-plugin_ctx_key_initialized=1 标识已经完成初始化

struct plugin_ctx_s {
  cdtime_t interval;
  cdtime_t flush_interval;
  cdtime_t flush_timeout;
};
typedef struct plugin_ctx_s plugin_ctx_t;

struct write_queue_s {
  value_list_t *vl;      采集的指标，对应types.db中的一行
  plugin_ctx_t ctx;      线程上下文，干嘛用的？？？
  write_queue_t *next;
};
typedef struct write_queue_s write_queue_t;

value_list_t


写入插件的回调函数接口：
typedef int (*plugin_write_cb)(const data_set_t *, const value_list_t *, user_data_t *);


typedef unsigned long long counter_t;
typedef double gauge_t;
typedef int64_t derive_t;
typedef uint64_t absolute_t;

union value_u {
  counter_t counter;
  gauge_t gauge;
  derive_t derive;
  absolute_t absolute;
};
typedef union value_u value_t;

struct value_list_s {
  value_t *values; 采集指标，该值类型与数量与types.db中的定义相同，插件填充
  size_t values_len; 采集指标数，同样与types.db中的值相同，插件填充
  cdtime_t time; 可以在plugin_value_list_clone()中自动填充
  cdtime_t interval; 同上
  char host[DATA_MAX_NAME_LEN]; 同上
  char plugin[DATA_MAX_NAME_LEN]; 从哪个插件获取，需要插件填充
  char plugin_instance[DATA_MAX_NAME_LEN];  同上，可选
  char type[DATA_MAX_NAME_LEN]; 根据该参数从data_sets中查找，需要插件填充
  char type_instance[DATA_MAX_NAME_LEN]; 同上，可选
  meta_data_t *meta; 元数据，用于插件特定目的使用，目前多数插件没有使用
};
typedef struct value_list_s value_list_t;




在 Collectd 中，其时间通过 cdtime_t 结构体表示，采用 64-bits 无符号整型，所有相关的处理都通过对应的函数处理，例如 读取时间 ```cf_util_get_cdtime()```、获取当前时间 ```cdtime()```、时间转换为浮点 ```CDTIME_T_TO_DOUBLE()``` 等等。

关于时间相关的其它辅助函数，可以查看 utils_time.c 文件中的内容。


target_notification.c


plugin_register_notification() 插件注册信息
 |-create_register_callback() 注册到list_notification链表中

plugin_dispatch_notification() 发送notify信息，很多插件都可以通过该函数发送消息


The Notification target can be used to create a notification with arbitrary text.
Synopsis

<Target "notification">
  Message "Oops, the %{type_instance} temperature is currently %{ds:value}!"
  Severity "WARNING"
</Target>



Placeholders

The following placeholders will be replaces by their respective values in the Message option:

    %{host}
    %{plugin}
    %{plugin_instance}
    %{type}
    %{type_instance}
    %{ds:name}
    Will be replaced by the current rate of of the data source "name". Because counter values need to be transformed to rates, using this target in the PreCache chain may yield unexpected results.









http://en.proft.me/2014/05/6/monitoring-cpu-ram-network-collectd/


https://collectd.org/wiki/index.php/Chains

https://mailman.verplant.org/pipermail/collectd/2017-April/thread.html


collectd 数据类型详解

http://blog.kankanan.com/article/collectd-6570636e7c7b578b8be689e3.html

https://collectd.org/wiki/index.php/Build_system

https://collectd.org/wiki/index.php/High_resolution_time_format

https://collectd.org/wiki/index.php/Release_process



plugin_dispatch_multivalue() memory




complex 主要是会在检查配置文件读取配置项时，判断是否需要注册各种回调函数。





## threshold 实现


plugin_register_complex_config()
 |-cf_register_complex() 会添加到complex_callback_head链表的尾部

在配置初始化函数 ut_config() 函数中，会注册两个主要的回调函数 ut_missing() 以及 ut_check_threshold() 两个函数；其插件规则保存在threshold_tree，一个自平衡二叉树(AVL)结构体。



ut_check_threshold()
 |-threshold_search()
 |-uc_get_rate()
 |-ut_check_one_threshold()
 |-ut_report_state()



thresholds解析







## 插件实现

SRC-TODO:
  1. plugin_read_thread()添加线程ID，方便调试。
  2. 从一个固定整点时间点开始，而非从启动时间开始计时。


https://collectd.org/wiki/index.php/Plugin_architecture
InnoDB: Error: page 570 log sequence number 7289495
InnoDB: is in the future! Current system log sequence number 5574939.
InnoDB: Your database may be corrupt or you may have copied the InnoDB
InnoDB: tablespace but not the InnoDB log files. See
InnoDB: http://dev.mysql.com/doc/refman/5.6/en/forcing-innodb-recovery.html
InnoDB: for more information.

日志显示是数据文件的 LSN 比 Redo Log 的 LSN 要大，当系统尝试使用 Redo Log 去修复数据页面的时候，发现 Redo Log LSN 比数据页面还小，所以导致错误。数据页的 LSN 在一般情况下，都是小于 Redo Log 的，因为在事物提交时先顺序写入 Redo Log ，然后后台线程将脏页刷新到数据文件中，所以，数据页的 LSN 正常情况下永远会比 Redo Log 的 LSN 小。

可能场景：
1. 如果有批量更新未提交，然后 kill 进程之后执行回滚操作，但是未等回滚执行完毕就 kill -9 mysql 导致回滚崩溃。

buf_page_io_complete() BP的异步读写完成
 |-buf_page_is_corrupted() 直接报错函数
   |-memcmp() 非压缩页，则检查页头部与尾部的LSN是否相同
   |-log_peek_lsn() 获取当前的log_sys->lsn，TODO如何获取？？？
   | <<<IMPORTANT>>> 如果当前LSN小于页面保存的FIL_PAGE_LSN那么则报错"in the future"
   |-buf_page_is_checksum_valid_crc32() 根据不同的页校验算法进行校验

页是否损坏判断：
1. 非压缩页校验 FIL_PAGE_LSN 和 FIL_PAGE_END_LSN_OLD_CHKSUM 所在的低 4 Bytes 是否相同，LSN是8Bytes为什么只校验低4Bytes？？？
2. 如果是

InnoDB 恢复流程
https://forums.cpanel.net/threads/innodb-corruption-repair-guide.418722/
https://boknowsit.wordpress.com/2012/12/22/mysql-log-is-in-the-future/


FIXME:
mysql-innodb-crash-recovery.html
recv_recovery_from_checkpoint_start() 函数末尾设置recv_lsn_checks_on为TRUE
 |-recv_find_max_checkpoint() 从redolog中...
 |-log_group_read_checkpoint_info()
collectd.html

增加 History 配置项，用于保存历史数据。


typedef unsigned long long counter_t;
typedef double gauge_t;
typedef int64_t derive_t;
typedef uint64_t absolute_t;

global cache

如果没有则通过 uc_insert() 写入，



Programs must be written for people to read, and only incidentally for machines to execute.

I'm not that familiar with automake, but I am pretty sure there is some way to solve this dilemma. Can anybody here give me a hint


?int uc_get_history(const data_set_t *ds, const value_list_t *vl, gauge_t *ret_history, size_t num_steps, size_t num_ds);

ds+vl 为调用 write 回调函数时的入参，



什么情况下会阻塞？？

fc_register_target()  添加到target_list_head链表的末尾

fc_configure()
 |-fc_init_once() 初始化内置target，包括了jump、stop、return、write
 |-fc_config_add_chain() 如果使用Chain参数
   |-fc_config_add_target()

fc_process_chain() 流程的处理

https://collectd.org/wiki/index.php/Target:Notification


fc_register_match() 添加到match_list_head链表的末尾
-->

## BUG-FIX

### 日志输出

如果将日志输出到文件时，可能会报如下的错误。

{% highlight text %}
collectd -C collectd.conf -t
logfile plugin: fopen (/opt/collectd/var/log/collectd.log) failed: No such file or directory
{% endhighlight %}

修改 logfile.c 中的默认文件，设置为标准错误输出 ```STDERR``` 。

{% highlight c %}
//#define DEFAULT_LOGFILE LOCALSTATEDIR "/log/collectd.log"
#define DEFAULT_LOGFILE "stderr"
{% endhighlight %}


## 参考

一些与 collectd 相关的工具，可以参考 [Related sites](http://collectd.org/related.shtml) 。

{% highlight text %}
{% endhighlight %}

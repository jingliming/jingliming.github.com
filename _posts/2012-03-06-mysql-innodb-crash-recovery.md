---
title: InnoDB 崩溃恢复
layout: post
comments: true
language: chinese
category: [mysql,database]
keywords: mysql,innodb,crash recovery,崩溃恢复
description:
---


<!-- more -->

## 崩溃恢复

InnoDB 的数据恢复是一个很复杂的过程，在其恢复过程中，需要 redolog、binlog、undolog 等参与，接下来具体了解下整个恢复的过程。

{% highlight text %}
innobase_init()
 |-innobase_start_or_create_for_mysql()
   |
   |-recv_sys_create()   创建崩溃恢复所需要的内存对象
   |-recv_sys_init()
   |
   |-srv_sys_space.check_file_spce()                检查系统表空间是否正常
   |-srv_sys_space.open_or_create()              1. 打开系统表空间，并获取flushed_lsn
   | |-read_lsn_and_check_flags()
   |   |-open_or_create()                           打开系统表空间
   |   |-read_first_page()                          读取第一个page
   |   |-buf_dblwr_init_or_load_pages()             将双写缓存加载到内存中，如果ibdata日志损坏，则通过dblwr恢复
   |   |-validate_first_page()                      校验第一个页是否正常，并读取flushed_lsn
   |   | |-mach_read_from_8()                       读取LSN，偏移为FIL_PAGE_FILE_FLUSH_LSN
   |   |-restore_from_doublewrite()                 如果有异常，则从dblwr恢复
   |
   |-log_group_init()                               redo log的结构初始化
   |-srv_undo_tablespaces_init()                    对于undo log表空间恢复
   |
   |-recv_recovery_from_checkpoint_start()       2. 从redo-log的checkpoint开始恢复；注意，正常启动也会调用
   | |-buf_flush_init_flush_rbt()                   创建一个红黑树，用于加速插入flush list
   | |                                              通过force_recovery判断是否大于SRV_FORCE_NO_LOG_REDO
   | |-recv_find_max_checkpoint()                   查找最新的checkpoint点，在此会校验redo log的头部信息
   | | |-log_group_header_read()                    读取512字节的头部信息
   | | |-mach_read_from_4()                         读取redo log的版本号LOG_HEADER_FORMAT
   | | |-recv_check_log_header_checksum()           版本1则校验页的完整性
   | | | |-log_block_get_checksum()                 获取页中的checksum，也就是页中的最后四个字节
   | | | |-log_block_calc_checksum_crc32()          并与计算后的checksum比较
   | | |-recv_find_max_checkpoint_0()
   | |   |-log_group_header_read()
   | |
   | |-recv_group_scan_log_recs()                3. 从checkpoint-lsn处开始查找MLOG_CHECKPOINT
   | | |-log_group_read_log_seg()
   | | |-recv_scan_log_recs()
   | |   |-recv_parse_log_recs()
   | |-recv_group_scan_log_recs()
   | |                                              ##如果flushed_lsn和checkponit lsn不同则恢复
   | |-recv_init_crash_recovery()
   | |-recv_init_crash_recovery_spaces()
   | |
   | |-recv_group_scan_log_recs()
   |
   |-trx_sys_init_at_db_start()
   |
   |-recv_apply_hashed_log_recs()                    当页LSN小于log-record中的LSN时，应用redo日志
   | |-recv_recover_page()                           实际调用recv_recover_page_func()
   |   |-recv_parse_or_apply_log_rec_body()
   |
   |-recv_recovery_from_checkpoint_finish()          完成崩溃恢复

fil_op_write_log() 些日志



fil_names_write()  写入MLOG_FILE_NAME
fil_name_write()
fil_op_write_log()

{% endhighlight %}

MLOG_FILE CHECKPOINT 如何写入？？？如何使用？？？kkkk

{% highlight text %}
flushed_lsn
  只有在系统表空间的第一页存在，偏移量为FIL_PAGE_FILE_FLUSH_LSN(26)，至少在此LSN之前的页已经刷型到磁盘；
  该LSN通过fil_write_flushed_lsn()函数写入；
{% endhighlight %}

1. 从系统表空间中读取flushed_lsn，每次刷脏时都会写入系统表空间的第一页，而且为了防止写入异常会使用Double Write Buffer；
2. 从redo log头部中读取两个checkpoint值，并比较获取最新的checkpoint信息；




mlog 如何记录的那些 ibd 时需要读取的。





InnoDB 在 MySQL 启动的时候，会对 redo-log 进行日志回放，通过 recv_sys_t 结构来进行数据恢复和控制的，它的结构如下：

{% highlight text %}
struct recv_sys_t {

mutex_t     mutex;                                 /*保护锁*/
ibool  apply_log_recs;                        /*正在应用log record到page中*/
ibool     apply_batch_on;                     /*批量应用log record标志*/

dulint  lsn;
ulint  last_log_buf_size;

byte*     last_block;                             /*恢复时最后的块内存缓冲区*/
byte*    last_block_buf_start;             /*最后块内存缓冲区的起始位置，因为last_block是512地址对齐的，需要这个变量记录free的地址位置*/
byte*   buf;                                        /*从日志块中读取的重做日志信息数据*/
ulint  len;    /*buf有效的日志数据长度*/

dulint    parse_start_lsn;                       /*开始parse的lsn*/
dulint   scanned_lsn;                           /*已经扫描过的lsn序号*/

ulint   scanned_checkpoint_no;          /*恢复日志的checkpoint 序号*/
ulint  recovered_offset;                       /*恢复位置的偏移量*/

dulint    recovered_lsn;                         /*恢复的lsn位置*/
dulint   limit_lsn;                                  /*日志恢复最大的lsn,暂时在日志重做的过程没有使用*/

ibool   found_corrupt_log;                   /*是否开启日志恢复诊断*/

log_group_t*  archive_group;

mem_heap_t*   heap;                             /*recv sys的内存分配堆,用来管理恢复过程的内存占用*/

  hash_table_t*   addr_hash;       // 以(space_id+page_no)为KEY的Hash表

ulint   n_addrs;                                        /*addr_hash中包含recv_addr的个数*/

};
{% endhighlight %}

在这个结构中，比较复杂的是addr_hash这个哈希表，这个哈希表是用sapce_id和page_no作为hash key,里面存储有恢复时对应的记录内容。

恢复日志在从日志文件中读出后，进行解析成若干个recv_t并存储在哈希表当中。在一个读取解析周期过后，日志恢复会对hash表中的recv_t中的数据写入到ibuf和page中。这里为什么要使用hash表呢？个人觉得是为了同一个page的数据批量进行恢复的缘故，这样可以page减少随机插入和修改。 以下是和这个过程相关的几个数据结构:


{% highlight text %}
/*对应页的数据恢复操作集合*/
struct recv_addr_t {
     ulint   state;          /*状态，RECV_NOT_PROCESSED、RECV_BEING_PROCESSED、RECV_PROCESSED*/
      ulint  space;         /*space的ID*/
       ulint     page_no;    /*页序号*/
  UT_LIST_BASE_NODE_T(recv_t) rec_list;  // 该页对应的log records地址
         hash_node_t     addr_hash;
};

/*当前的记录操作*/
struct recv_t {
     byte    type;             /*log类型*/
      ulint  len;               /*当前记录数据长度*/
       recv_data_t* data;    /*当前的记录数据list*/

  lsn_t  start_lsn;                 // mtr起始LSN
  lsn_t  end_lsn;                   // mtr结尾LSN
  UT_LIST_NODE_T(recv_t)  rec_list; // 该页对应的log records
};

struct recv_data_t {
  recv_data_t*   next;  // 指向下个结构体，该地址之后为一大块内存，用于存储log record消息体
};
{% endhighlight %}

他们的内存关系结构图如下： \
2.重做日志推演过程的LSN关系
除了这个恢复的哈希表以外，recv_sys_t中的各种LSN也是和日志恢复有非常紧密的关系。以下是各种lsn的解释： parse_start_lsn 本次日志重做恢复起始的lsn，如果是从checkpoint处开始恢复，等于checkpoint_lsn。 scanned_lsn 在恢复过程，将恢复日志从log_sys->buf解析块后存入recv_sys->buf的日志lsn. recovered_lsn 已经将数据恢复到page中或者已经将日志操作存储addr_hash当中的日志lsn; 在日志开始恢复时：
parse_start_lsn = scanned_lsn = recovered_lsn = 检查点的lsn。
在日志完成恢复时:
parse_start_lsn = 检查点的lsn
scanned_lsn = recovered_lsn = log_sys->lsn。
在日志推演过程中lsn大小关系如下：
\
3.日志恢复的主要接口和流程
恢复日志主要的接口函数: recv_recovery_from_checkpoint_start 从重做日志组内的最近的checkpoint开始恢复数据
recv_recovery_from_checkpoint_finish 结束从重做日志组内的checkpoint的数据恢复操作
recv_recovery_from_archive_start 从归档日志文件中进行数据恢复
recv_recovery_from_archive_finish 结束从归档日志中的数据恢复操作
recv_reset_logs 截取重做日志最后一段作为新的重做日志的起始位置，可能会丢失数据。
重做日志恢复数据的流程(checkpoint方式) 1.当MySQL启动的时候，先会从数据库文件中读取出上次保存最大的LSN。
2.然后调用recv_recovery_from_checkpoint_start，并将最大的LSN作为参数传入函数当中。
3.函数会先最近建立checkpoint的日志组，并读取出对应的checkpoint信息
4.通过checkpoint lsn和传入的最大LSN进行比较，如果相等，不进行日志恢复数据，如果不相等，进行日志恢复。
5.在启动恢复之前，先会同步各个日志组的archive归档状态
6.在开始恢复时，先会从日志文件中读取2M的日志数据到log_sys->buf，然后对这2M的数据进行scan,校验其合法性，而后将去掉block header的日志放入recv_sys->buf当中，这个过程称为scan,会改变scanned lsn.
7.在对2M的日志数据scan后，innodb会对日志进行mtr操作解析，并执行相关的mtr函数。如果mtr合法，会将对应的记录数据按space page_no作为KEY存入recv_sys->addr_hash当中。
8.当对scan的日志数据进行mtr解析后，innodb对会调用recv_apply_hashed_log_recs对整个recv_sys->addr_hash进行扫描，并按照日志相对应的操作进行对应page的数据恢复。这个过程会改变recovered_lsn。
9.如果完成第8步后，会再次从日志组文件中读取2M数据，跳到步骤6继续相对应的处理，直到日志文件没有需要恢复的日志数据。
10.innodb在恢复完成日志文件中的数据后，会调用recv_recovery_from_checkpoint_finish结束日志恢复操作，主要是释放一些开辟的内存。并进行事务和binlog的处理。
上面过程的示意图如下：




## undrop tools


### stream_parser

stream_parser 用于在字节流中查找 InnoDB 页，可以解析多种文件，包括了 ibdata1、*.ibd、raw partition；其中 raw partitioin 可以通过 ```df -h /var/lib/mysql``` 查看。

{% highlight text %}
stream_parser -f /dev/mapper/vg_centos-lv_root -t 50g
选项：
-h    打印帮助信息；
-f    必选项，用于指定文件，可以为 ibdata1、table.ibd、/dev/sda1；
-s    指定缓存大小，例如1G、100M(默认值)、10K；
-t    指定扫描大小；
-d    指定输出的目录；
-V,-g 打印调试信息；
-T
{% endhighlight %}


{% highlight text %}
main()
 |-open_ibfile()
 |-process_ibfile()
   |-lseek64()                  多进程处理时，会将文件分块处理
   |-read()                     将文件读取到缓存中
   |-valid_blob_page()          校验
   |-valid_innodb_page()
   |-process_ibpage()           处理页
{% endhighlight %}

### c_parser

c_parser 用于解析 InnoDB 页，并将其中的数据解析出来，因为 InnoDB 不包含表的结构信息，所以需要告诉 c_parser 解析的表结构信息。

https://twindb.com/undrop-tool-for-innodb/

https://twindb.com/tag/stream_parser/

https://github.com/chhabhaiya/undrop-for-innodb

## RedoLog

InnoDB 有 Buffer Pool 也就是数据库的页面缓存，对数据页的任何修改都会先在 BP 上修改，然后这样的页面将被标记为 dirty 并被放到专门的 flush list 上，后续将由 master thread 或专门的刷脏线程阶段性的将这些页面写入磁盘。

这样带来的好处是避免每次写操作都会导致大量的随机 IO，阶段性的刷脏可以将多次对页面的修改合并成一次 IO 操作，同时异步写入也降低了访问的时延。

然而，如果在脏页未刷入磁盘时，服务器或者进程非正常关闭，将会导致这些修改操作丢失，如果写入操作正在进行，甚至会由于损坏数据文件导致数据库不可用。

为了避免上述问题，InnoDB 会将所有对页面的修改操作写入一个特定的文件，也就是 redolog，并在数据库启动时从此文件进行恢复操作。这样推迟了 BP 页面的刷脏，提升了数据库的吞吐，有效的降低了访问时延；带来的问题是额外的写 redo log 操作的开销 (顺序 IO 速度较快)，以及数据库启动时恢复操作所需的时间。

接下来将结合 MySQL 代码看下 Log 文件的结构、生成过程以及数据库启动时的恢复流程。

### LSN

LSN(Log Sequence Number，日志序列号，ib_uint64_t) 保存在 log_sys.lsn 中，在 log_init() 中初始化，初始值为 LOG_START_LSN(8192) 。改值实际上对应日志文件的偏移量，新的 LSN＝旧的LSN + 写入的日志大小，在调用日志写入函数时，LSN 就一直随着写入的日志长度增加。

{% highlight text %}
#define OS_FILE_LOG_BLOCK_SIZE      512
#define LOG_START_LSN       ((lsn_t) (16 * OS_FILE_LOG_BLOCK_SIZE))
{% endhighlight %}

日志通过 log_write_low()@log/log0log.c 函数写入。

{% highlight cpp %}
void log_write_low(byte* str, ulint str_len)
{
    log_t* log = log_sys;
part_loop:
    ... ...                                        // 计算写入日志的长度
    ut_memcpy(log->buf + log->buf_free, str, len); // 将日志内容拷贝到log buffer
    ... ...
    log->lsn += len;
}
{% endhighlight %}

如上所述，LSN 是不会减小的，它是日志位置的唯一标记，在重做日志写入、checkpoint 构建和 PAGE 头里面都有 LSN。

例如当前重做日志的 LSN=2048，这时候调用 log_write_low() 写入一个长度为 700 的日志，2048 刚好是 4 个 block 长度，那么需要存储 700 长度的日志，需要两个 block(单个block只能存496个字节)，那么很容易得出新的 LSN 为。

{% highlight text %}
LSN=2048+700+2*LOG_BLOCK_HDR_SIZE(12)+LOG_BLOCK_TRL_SIZE(4)=2776
{% endhighlight %}

<!--
lsn是联系dirty page、redo log record和redo log file的纽带，每个redo log record被拷贝到内存的log buffer时会产生一个相关联的lsn，而每个页面修改时会产生一个log record，从而每个数据库的page也会有一个相关联的lsn，这个lsn记录在每个page的header字段中。为了保证WAL（Write-Ahead-Logging）要求的逻辑，dirty page要求其关联lsn的log record已经被写入log file才允许执行flush操作。<br><br>

从上面的结构定义可以看出有很多LSN相关的定义，那么这些LSN直接的关系是怎么样的呢？理解这些LSN之间的关系对理解整个重做日志系统的运作机理会有极大的信心。以下各种LSN的解释：
<ul><li>
lsn<br>
当前log系统最后写入日志时的LSN。</li><br><li>

flush_lsn<br>
redo-log buffer最后一次数据刷盘数据末尾的LSN，作为下次刷盘的起始LSN。</li><br><li>

written_to_some_lsn<br>
单个日志组最后一次日志刷盘时的起始LSN。</li><br><li>

written_to_all_lsn<br>
所有日志组最后一次日志刷盘是的起始LSN。

 last_checkpoint_lsn        最后一次建立checkpoint日志数据起始的LSN

 next_checkpoint_lsn        下一次建立checkpoint的日志    数据起始的LSN,用log_buf_pool_get_oldest_modification获得的

 archived_lsn               最后一次归档日志数据起始的LSN
 next_archived_lsn          下一次归档日志数据的其实LSN
</li></ul>



b)        文件为顺序写入，当达到最后一个文件末尾时，会从第一个文件开始顺序复用。

c)        lsn: Log Sequence Number，是一个递增的整数。 Ib_logfile中的每次写入操作都包含至少1个log，每个log都带有一个lsn。在内存page修复过程中，只有大于page_lsn的log才会被使用。

d)        lsn的保存在全局变量log_sys中。递增数值等于每个log的实际内容长度。即如果新增的一个log长度是len，则log_sys->lsn += len.

e)        ib_logfile每次写入以512（OS_FILE_LOG_BLOCK_SIZE）字节为单位。实际写入函数 log_group_write_buf (log/log0log.c)

f)         每次写盘后是否flush，由参数innodb_flush_log_at_trx_commit控制。

在 innodb 中，维护了一个全局变量 struct log_struct log_sys ，
written_to_all_lsn
n表示实际已经写盘的lsn。需要这个值是因为并非每次生成log后就写盘。
flushed_to_disk_lsn
表示刷到磁盘的lsn。需要这个值是因为并非每次写盘后就flush。
buf
待写入的内容保存在buf中
buf_size
buf的大小。由配置中innodb_log_buffer_size决定，实际大小为innodb_log_buffer_size /16k * 16k。
buf_next_to_write
buf中下一个要写入磁盘的位置
buf_free
buf中实际内容的最后位置。当buf_free> buf_next_to_write时，说明内存中还有数据未写盘。



log 采用循环写法，
有三种情况――overflow， checkpoint和commit――可以导致写盘操作
如果data buffer满了，InnoDB将最近使用的buffer写入到数据库中，但是不可能足够的快。这种情况下，页头的LSN就起作用了。第一，InnoDB检查它的LSN是否比log文件中最近的log记录的LSN大，只有当log赶上了data的时候，才会将数据写到磁盘。换句话说，数据页不会写盘，直到相应的log记录需要写盘的时候。这就是先写日志策略。


-->

### 变量设置

简单来说 InnoDB 通过两个核心参数 ```innodb_buffer_pool_size```、```innodb_log_file_size```，分别定义了数据缓存和 redolog 的大小，而后者的大小也决定了 BP 中可以有多少脏页。当然，也不能因此就增大 redolog 文件的大小，这样，可能会导致系统启动时 Crash Recovery 时间增大。

redolog 保存在 ```innodb_log_group_home_dir``` 参数指定的目录下，文件名为 ib_logfile\*；undolog 保存在共享表空间 ibdata\* 文件中。

redolog 由一组固定大小的文件组成，顺序写入，而且文件循环使用，文件名为 ib_logfileN (其中N为从0开始的数字)，可以通过 ```innodb_log_file_size``` 和 ```innodb_log_files_in_group``` 参数控制文件的大小和数目，日志总大小为两者之积。

### 文件结构

先明确下常用概念，redolog 写入通常是以 ```OS_FILE_LOG_BLOCK_SIZE``` (512字节，编译时设置) 为单位顺序写入文件，每个 LogBlock 包含了一个 header 段、一个 tailer 段、一组 LogRecords；每条记录都有自己的 LSN，表示从日志记录创建开始到特定的日志记录已经写入的字节数。

头部信息，在 log0log.h 文件中定义，简单看下常用的保存字段。

{% highlight text %}
LOG_HEADER_FORMAT
  之前为LOG_GROUP_ID，用于标示redolog的分组ID，但是现在只支持一个分组；
LOG_HEADER_START_LSN
  日志文件记录的初始LSN，占用8字节，注意其偏移量与老版本有所区别；
LOG_HEADER_CREATOR
  备份程序会将填写备份程序和创建时间，通常是使用xtrabackup、mysqlbackup时会填充；
LOG_CHECKPOINT_1/2
  在头文件中定义的checkpoint位置，注意只有日志组了的第一文件会有该记录，每次checkpoint都会更新这两个值；
{% endhighlight %}

块的宏偏移量同样在 log0log.h 文件中定义，其中头部包括了 LOG_BLOCK_HDR_SIZE(12) 个字节，详细如下：

{% highlight text %}
LOG_BLOCK_HDR_NO
  四字节标示这是第几个block块，该值是通过LSN计算得来，详见log_block_convert_lsn_to_no()函数；
LOG_BLOCK_HDR_DATA_LEN
  两字节表示该block中已经有多少字节被使用；
LOG_BLOCK_FIRST_REC_GROUP
  两个字节表示该block中作为一个新的MTR开始log
LOG_BLOCK_CHECKPOINT_NO
  四个字节表示该block的checkpoint，也就是log_sys->next_checkpoint_no；
LOG_BLOCK_CHECKSUM
  四个字节记录了该block的校验值，通过innodb_checksum_algorithm变量指定算法；
{% endhighlight %}


innodb_fast_shutdown

innodb_force_recovery
innodb_force_recovery_crash

????LSN对应了日志文件的偏移量，为了减小故障恢复时间，引入了Checkpoint机制，

InnoDB在启动时会自动检测InnoDB数据和事务日志是否一致，是否需要执行相应的操作？？？保证数据一致性；当然，故障恢复时间与事务日志的大小相关。


在此涉及到两块内存缓冲，涉及到mtr/log_sys等内部结构，后续会一一介绍。？？？


Log记录生成

### 数据结构

首先看下 InnoDB 在内存中保存的一个全局变量，也就是 ```log_t* log_sys=NULL;``` 定义，其维护了一块称为 log buffer 的全局内存区域，也就是 ```log_sys->buff```，同时维护有若干 lsn 值等信息表示 logging 进行的状态；会在 ```log_init()``` 中对所有的内部区域进行分配并对各个变量进行初始化。

变量 log_sys 是 InnoDB 日志系统的中枢及核心对象，控制着日志的拷贝、写入、checkpoint 等核心功能，是连接 InnoDB 日志文件及 log buffer 的枢纽。

接下来，简单介绍下 log_t 中比较重要的字段值：

{% highlight cpp %}
struct log_t{
  char        pad1[CACHE_LINE_SIZE];
  lsn_t       lsn;                    // 接下来将要生成的log record使用此lsn的值；
  ulint       buf_free;   /*!< first free offset within the log
                  buffer in use */
  byte*       buf;                    // 全局的log buffer，？？？？？
  lsn_t       flushed_to_disk_lsn;    // 已经刷新到磁盘的LSN值，小于该值的日志已经被安全地记录到了磁盘上

  UT_LIST_BASE_NODE_T(log_group_t)    // 日志组，当前版本仅支持一组日志
          log_groups;                 // 包含了当前日志组的文件个数、每个文件的大小、space id等信息

  lsn_t       log_group_capacity;     // 当前日志文件的总容量在log_calc_max_ages()中计算，
                                      // (redo-log文件大小-头部)*0.9，其中乘0.9是一个安全系数

  lsn_t       max_modified_age_async;
                  /*!< when this recommended
                  value for lsn -
                  buf_pool_get_oldest_modification()
                  is exceeded, we start an
                  asynchronous preflush of pool pages */
  lsn_t       max_modified_age_sync;
                  /*!< when this recommended
                  value for lsn -
                  buf_pool_get_oldest_modification()
                  is exceeded, we start a
                  synchronous preflush of pool pages */
  lsn_t       max_checkpoint_age_async;
                  /*!< when this checkpoint age
                  is exceeded we start an
                  asynchronous writing of a new
                  checkpoint */
  lsn_t       max_checkpoint_age;
                  /*!< this is the maximum allowed value
                  for lsn - last_checkpoint_lsn when a
                  new query step is started */
  ib_uint64_t next_checkpoint_no;
                  /*!< next checkpoint number */
  lsn_t       last_checkpoint_lsn;
                  /*!< latest checkpoint lsn */
  lsn_t       next_checkpoint_lsn;
                  /*!< next checkpoint lsn */

{% endhighlight %}

上述结构体中的很多值会在初始化的时候进行设置，调用流程如下。

{% highlight text %}
innobase_start_or_create_for_mysql()
 |-log_group_init()
   |-log_calc_max_ages()
{% endhighlight %}

{% highlight cpp %}
static bool log_calc_max_ages(void)
{
    ... ...
    group = UT_LIST_GET_FIRST(log_sys->log_groups);
    smallest_capacity = LSN_MAX;
    while (group) {
        if (log_group_get_capacity(group) < smallest_capacity) {
            smallest_capacity = log_group_get_capacity(group);
        }
        group = UT_LIST_GET_NEXT(log_groups, group);
    }
    smallest_capacity = smallest_capacity - smallest_capacity / 10; /* Add extra safety */

    /* For each OS thread we must reserve so much free space in the
    smallest log group that it can accommodate the log entries produced
    by single query steps: running out of free log space is a serious
    system error which requires rebooting the database. */
    free = LOG_CHECKPOINT_FREE_PER_THREAD * (10 + srv_thread_concurrency)
        + LOG_CHECKPOINT_EXTRA_FREE;
    if (free >= smallest_capacity / 2) {
        success = false;
        goto failure;
    } else {
        margin = smallest_capacity - free;
    }

    margin = margin - margin / 10;	/* Add still some extra safety */

    log_sys->log_group_capacity = smallest_capacity;

    log_sys->max_modified_age_async = margin
        - margin / LOG_POOL_PREFLUSH_RATIO_ASYNC;
    log_sys->max_modified_age_sync = margin
        - margin / LOG_POOL_PREFLUSH_RATIO_SYNC;

    log_sys->max_checkpoint_age_async = margin - margin
        / LOG_POOL_CHECKPOINT_RATIO_ASYNC;
    log_sys->max_checkpoint_age = margin;

    ... ...
    return(success);
}
{% endhighlight %}




log_sys->buf_free
 
写入buffer的起始偏移量


log_sys->write_lsn
 当前正在执行的写操作使用的临界lsn值；
log_sys->current_flush_lsn
 当前正在执行的write + flush操作使用的临界lsn值，一般和log_sys->write_lsn相等；

log_sys->buf_next_to_write
 
buffer中还未写到log file的起始偏移量。下次执行write+flush操作时，将会从此偏移量开始
log_sys->max_buf_free
 
确定flush操作执行的时间点，当log_sys->buf_free比此值大时需要执行flush操作，具体看log_check_margins函数




lsn是联系dirty page，redo log record和redo log file的纽带。在每个redo log record被拷贝到内存的log buffer时会产生一个相关联的lsn，而每个页面修改时会产生一个log record，从而每个数据库的page也会有一个相关联的lsn，这个lsn记录在每个page的header字段中。为了保证WAL（Write-Ahead-Logging）要求的逻辑，dirty page要求其关联lsn的log record已经被写入log file才允许执行flush操作。







mtr(mini-transactions)在代码中对应了strunct mtr_t结构体，内部有一个局部buffer，会将一组log record集中起来，批量写入log buffer；mtr_t的结构体如下所示：

struct mtr_t {
  struct Impl {
    mtr_buf_t  m_memo;  由此mtr涉及的操作所造成的脏页列表，会在mtr_commit执行后添加到flush-list中，详见mtr_memo_pop_all()函数；
    mtr_buf_t  m_log;   mtr的局部缓存，记录log-records；
  };
};

创建一个mtr_t类型的对象；
执行mtr_start()初始化mtr_t的字段，包括local buffer；
在对BP中的Page进行修改的同时，调用mlog_write_ulint()、mlog_write_string()、mlog_write_ull()类似的函数，生成redo log record并保存在local buffer中；
执行mtr_commit()将local buffer中的redo log拷贝到全局的log_sys->buffer，同时将脏页添加到flush list，供后续执行flush操作时使用；








binlog_filter->add_ignore_db



####### 崩溃恢复(Crash Recovery)
Crash Recovery的起点，Checkpoint LSN存储位置？
InnoDB如何完成Redo日志的重做？
InnoDB如何定位哪些事务需要Rollback？
Crash Recovery需要等待Rollbach完成吗？
InnoDB各版本，在Crash Recovery流程上做了哪些优化？
mysqld_safe是否存在自动重启功能？

ha_recover


af_get_pct_for_lsn()
 |-log_get_max_modified_age_async()
  
MySQL · 引擎特性 · InnoDB 崩溃恢复过程
http://mysql.taobao.org/monthly/2015/06/01/

Database was not shutdown normally!   # InnoDB开始Crash Recovery{recv_init_crash_recovery_spaces()}
Starting crash recovery.




Doing recovery: scanned up to log sequence number 0            # 扫描redolog日志recv_scan_log_recs()




Starting an apply batch of log records to the database...      # 开始应用redolog recv_apply_hashed_log_recs()   
InnoDB: Progress in percent: 2 3 4 5 ... 99
Apply batch completed


1. 读取Checkpoint LSN
2. 从Checkpoint LSN开始向前遍历Redo Log File
   重做从Checkpoint LSN开始的所有Redo日志
3. 重新构造系统崩溃时的事务
   Commit事务，等待Purge线程回收
   Prepare事务，由MySQL Server控制提交或者回滚(与Binlog 2PC相关)
   Active事务，回滚
4. 新建各种后台线程，Crash Recovery完成返回


正常关闭时，会在flush redo log和脏页后，做一次完全同步的checkpoint，并将checkpoint的LSN写到第一个ibdata文件的第一个page中，详细可以参考fil_write_flushed_lsn()。


innodb_counter_info[] 定义了监控计数器

http://mysqllover.com/?p=376
http://hedengcheng.com/?p=183
http://mysql.taobao.org/monthly/2015/05/01/
http://mysql.taobao.org/monthly/2016/05/01/
http://tech.uc.cn/?p=716
http://hedengcheng.com/?p=88InnoDB
http://mysqllover.com/?p=620
http://apprize.info/php/effective/6.html
http://www.cnblogs.com/chenpingzhao/p/5107480.html
https://www.xaprb.com/blog/2011/01/29/how-innodb-performs-a-checkpoint/
数据库内核分享
https://www.slideshare.net/frogd/inno-db-15344119

检查保存到磁盘的最大checkpoint LSN与redo-log的LSN是否一致；


事务源码
崩溃恢复
ReadView
关闭过程
Undo-Redo


https://blogs.oracle.com/mysqlinnodb/entry/repeatable_read_isolation_level_in


## 崩溃恢复

第一步操作
停止、备份、重启

1. 停止
停止MySQL服务，如果已经下线或者崩溃可以忽略，目的是要冻结数据和表文件的当前状态保证没有新数据写入，可以直接进行文件操作，不需要关心是否会导致数据不一致或者数据丢失等。
/etc/init.d/mysqld stop

2. 备份
备份整个数据目录，也可以只备份数据和日志文件。
mkdir /data/innodb.bak
cd /var/lib/mysql
dd if=ibdata1 of=ibdata1.bak conv=noerror
cp -p ./ibdata* /data/innodb.bak/
cp -p ./ib_log* /data/innodb.bak/
里面需要注意的是，对于ibdata1分别使用了dd和cp进行复制，这是由于两者复制方式不同，dd是直接复制原始文件，而cp则复制文件内容到一个新的文件；可能这个不是必须的，但至少不是一个坏习惯。

如果没有数据备份，也可以直接备份整个数据目录；当然，这一步骤会比较耗时，对于紧急情况不太实际，如果这不可行，至少数据文件和InnoDB数据库目录应该提供一些能回退的数据：
cp -Rp /var/lib/mysql{,.orig}

3. 备份InnoDB数据库文件
如上所述，如果没有备份完整的MySQL数据目录，最好还是备份下InnoDB对应的目录，如果不确认哪些目录下含有InnoDB表，可以直接使用如下命令。
DATADIR=/var/lib/mysql; find $DATADIR -type f -name *.ibd | awk -F/ '{print $(NF-1)}' | sort | uniq | xargs -I {} cp -Rp $DATADIR/{} /root/innodb.bak

4. 重启MySQL服务
如果启动不会导致崩溃的话，可以启动然后通过mysqldump备份数据。
/etc/init.d/mysqld start
mysqldump --single-transaction -AER > /root/dump_wtrans.sql
mysqldump -AER > /root/dump.sql
备份之后，一定要检查是否可用，有可能会由于部分情况导致数据没有备份下来。

5. 无法启动
如果无法启动或者启动后无法备份，除了希望尽快上线之外，启动服务的目的之一是尽可能的恢复数据，减小数据丢失。实际上，InnoDB为了满足ACID属性，保持数据一致性，如果遇到数据的任何问题，它几乎总是使MySQL崩溃以防止进一步损坏数据。

可以通过如下方式修改innodb_force_recovery参数：
mode=1; sed -i "/^\[mysqld\]/{N;s/$/\ninnodb_force_recovery=$mode/}" /etc/my.cnf
然后，一旦你准备把你的服务器返回到默认模式，你可以通过以下命令删除innodb_force_recovery行：
sed -i '/innodb_force_recovery/d' /etc/my.cnf

Mode 1当遇到损坏页时，不使 MySQL 崩溃
Mode 2不运行后台操作
Mode 3不会尝试回滚事务
Mode 4不计算统计数据或应用存储/缓冲的变化
Mode 5在启动过程中不查看撤消日志
Mode 6在启动时不从重做日志（ib_logfiles）前滚
  
第二步 判断问题所在

1. 检查日志
如果怀疑InnoDB表或数据库损坏，可能是因为数据页损坏、数据不存在、MySQL服务无法启动等，对于任何一种情况，你要首先查看的是MySQL错误日志。
tail -200 /var/lib/mysql/`hostname`.err
通常可以大致确认服务崩溃的原因，在此仅讨论三种比较常见的情况：A)页损坏；B)LSN；C)数据字典。

1.A 页损坏
InnoDB: Database page corruption on disk or a failed
InnoDB: file read of page 515891.
通常会包含了相关的一些信息，例如通过调用堆栈确认是在什么场景下触发的异常、崩溃时的环境参数，重点查看下是那个页损坏了，当然也有可能只是由于某些原因无法读取而已。

当然，崩溃后并不一定意味着真正有页损坏，事实上，在某些情况下，这可能只是操作系统的文件缓存被损坏；所以，建议在创建备份后，在无任何进一步操作之前，可以通过如下的工具检查是否有页损坏，或者直接重启你的计算机看下是否有异常。
#!/bin/bash
for i in $( ls /var/lib/mysql/*/*.ibd); do
    innochecksum $i
done
innochecksum工具会查看表空间文件中的页，计算每页的校验值；然后，将计算值与存储的校验相比，如果两个值不同，通常意味着页被损坏，则报错。
如果MySQL是在线且可访问的，你可以使用CHECK TABLE命令。

2.B LSN/时间异常
120901  9:43:55  InnoDB: Error: page 70944 log sequence number 8 1483471899
InnoDB: is in the future! Current system log sequence number 5 612394935.
InnoDB: Your database may be corrupt or you may have copied the InnoDB
InnoDB: tablespace but not the InnoDB log files.
在InnoDB上每次修改操作，都会生成一个redo记录，然后保存在ib_logfileN文件中，这些记录会按顺序写入文件，如果写满则重新开始；所有这些记录都有一个相关的LSN，通常也就是日志文件的偏移量。

？？？？？？
此外，当一个数据库被修改，在该数据库中的特定页面也得到一个相关LSN。两者之间，这些LSN被一起检查，确保操作以正确的顺序执行。LSN本身基本上是一个到日志文件的偏移，且存储在数据库页头中的LSN告诉InnoDB有多少日志需要被刷。
在过程中，无论是意外重启，内存问题，文件系统损坏，复制问题，手动更改为InnoDB的文件或其他，这些LSN不再“同步”。无论是否使你的服务器崩溃，这应该被当作合理损坏，通常你需要解决它。

2.C 数据字典错误
[ERROR] Table ./database/table has no primary key in InnoDB data dictionary, but has one in MySQL!
InnoDB: Error: table ‘database/table’
InnoDB: in InnoDB data dictionary has tablespace id 423,
InnoDB: but tablespace with that id or name does not exi st. Have
InnoDB: you deleted or moved .ibd files
?[ERROR] Cannot find or open table database/table from
the internal data dictionary of InnoDB though the .fr m file for the
table exists. Maybe you have deleted and recreated InnoDB data
files but have forgotten to delete the corresponding .frm file s
of InnoDB tables, or you have moved .frm files to another datab ase
?or, the table contains indexes that this version of the engine
doesn’t support.
InnoDB的数据字典存在于系统表空间(Space0)，保存在ibdata1文件中，储存了包括表、列、索引的元数据信息；与每个InnoDB表的*.frm文件有很多相同信息。如果由于某些原因导致ibdata1文件被损坏、修改、移动或或替换，就可能导致无法通过数据字典查看数据。

？？？？
如果你看过之前的错误描述，你应该知道在ibdata1中（或以其他方式命名）文件中的数据与在单个表空间/.ibd / .frm文件的数据之间有明显的关联。当该关联丢失或损坏，可能会发生不好的情况。所以这像这样的数据字典的错误出现，最常见的原因是有些文件被手动移动或修改。它通常归结为：“数据字典预计这一文件或表空间在这里，但它不在！”，或“.ibd / .frm文件预计此项目在数据字典中，但它不在！ “。再次记住，数据字典存储在ibdata文件中，在大多数环境中，就是MySQL数据目录中的ibdata1。
 
  

MySQL InnoDB Corruption Repair Guide
https://forums.cpanel.net/threads/innodb-corruption-repair-guide.418722/
http://www.askmaclean.com/archives/mysql-recover-innodb.html



GTID的全称为 global transaction identifier  ， 可以翻译为全局事务标示符，GTID在原始master上的事务提交时被创建。GTID需要在全局的主-备拓扑结构中保持唯一性，GTID由两部分组成：
GTID = source_id:transaction_id

source_id用于标示源服务器，用server_uuid来表示，这个值在第一次启动时生成，并写入到配置文件data/auto.cnf中
transaction_id则是根据在源服务器上第几个提交的事务来确定。

一个GTID的生命周期包括：
1.事务在主库上执行并提交
给事务分配一个gtid（由主库的uuid和该服务器上未使用的最小事务序列号），该GTID被写入到binlog中。
2.备库读取relaylog中的gtid，并设置session级别的gtid_next的值，以告诉备库下一个事务必须使用这个值
3.备库检查该gtid是否已经被其使用并记录到他自己的binlog中。slave需要担保之前的事务没有使用这个gtid，也要担保此时已分读取gtid，但未提交的事务也不恩呢过使用这个gtid.
4.由于gtid_next非空，slave不会去生成一个新的gtid，而是使用从主库获得的gtid。这可以保证在一个复制拓扑中的同一个事务gtid不变。

由于GTID在全局的唯一性，通过GTID，我们可以在自动切换时对一些复杂的复制拓扑很方便的提升新主库及新备库，例如通过指向特定的GTID来确定新备库复制坐标。

当然，使用GTID也有一些限制：
1.事务中的更新包含非事务性存储引擎，这可能导致多个GTID分配给同一个事务。
2. create table…select语句不被支持，因为该语句会被拆分成create table 和insert两个事务，并且这个两个事务被分配了同一个GTID，这会导致insert被备库忽略掉。
3.不支持CREATE/DROP临时表操作

可以看到，支持GTID的复制对一些语句都有一些限制，MySQL也提供了一个选项disable-gtid-unsafe-statements以禁止这些语句的执行。




### ibd页损坏
Page损坏的情况比较多：二级索引页损坏(可以通过OPTIMIZE TABLE恢复)；聚集索引页损坏；表字典损坏；
----- 使用VIM编译
vim -b titles.ibd
----- 通过外部xxd程序改变，注意修改完之后一定要通过-r选项恢复为二进制格式
:%!xxd
:%!xxd -r
当执行check table titles;检查表是否正常时，就会由于页面错误退出，报错信息如下。

buf_page_io_complete()
2017-03-09T08:58:34.750125Z 4 [ERROR] InnoDB: Database page corruption on disk or a failed file read of page [page id: space=NUM, page number=NUM]. You may have to recover from a backup.
len 16384; hex ... ...
InnoDB: End of page dump
InnoDB: Page may be an index page where index id is 48
2017-03-09T08:58:34.804632Z 4 [ERROR] [FATAL] InnoDB: Aborting because of a corrupt database page in the system tablespace. Or,  there was a failure in tagging the tablespace  as corrupt.
2017-03-09 16:58:34 0x7fe6e87b7700  InnoDB: Assertion failure in thread 140629719611136 in file ut0ut.cc line 916

实际上是可以正常重启的，但是一旦查询到该页时，仍然会报错，接下来看看如何恢复其中还完好的数据。

修改my.ini中的innodb_force_recovery参数，默认是0，此时可以修改为1-6，使mysqld在启动时跳过部分恢复步骤，在启动后将数据导出来然后重建数据库；当然，不同的情况可能恢复的数据会有所不同。

1. SRV_FORCE_IGNORE_CORRUPT: 忽略检查到的corrupt页；**
2. SRV_FORCE_NO_BACKGROUND): 阻止主线程的运行，在srv_master_thread()中处理；**

3. SRV_FORCE_NO_TRX_UNDO):不执行事务回滚操作。
4. SRV_FORCE_NO_IBUF_MERGE):不执行插入缓冲的合并操作。
5. SRV_FORCE_NO_UNDO_LOG_SCAN):不查看重做日志，InnoDB存储引擎会将未提交的事务视为已提交。
6. SRV_FORCE_NO_LOG_REDO):不执行redo log。

另外，需要注意，设置参数值大于0后，可以对表进行select,create,drop操作，但insert,update或者delete这类操作是不允许的。

再次执行check table titles;时，会报如下的错误。
+------------------+-------+----------+---------------------------------------------------+
| Table            | Op    | Msg_type | Msg_text                                          |
+------------------+-------+----------+---------------------------------------------------+
| employees.titles | check | Warning  | InnoDB: The B-tree of index PRIMARY is corrupted. |
| employees.titles | check | error    | Corrupt                                           |
+------------------+-------+----------+---------------------------------------------------+
2 rows in set (2.47 sec)

SELECT * FROM titles INTO OUTFILE '/tmp/titles.csv'
   FIELDS TERMINATED BY ',' ENCLOSED BY '"'
   LINES TERMINATED BY '\r\n';
注意上述在使用LOAD DATA INFILE或者SELECT INTO OUTFILE操作时，需要在配置文件中添加secure_file_priv=/tmp配置项，或者配置成secure_file_priv="/"不限制导入和导出路径。
如何确认哪些数据丢失了？？？？

http://www.runoob.com/mysql/mysql-database-export.html

如下是导致MySQL表毁坏的常见原因：
1、 服务器突然断电或者强制关机导致数据文件损坏；
2、 mysqld进程在修改表时被强制杀掉，例如kill -9；
3、 磁盘故障、服务器宕机等硬件问题无法恢复；
4、 使用myisamchk的同时，mysqld也在操作表；
5、 MySQL、操作系统、文件系统等软件的bug。




表损坏的典型症状：
 　　　　1 、当在从表中选择数据之时，你得到如下错误： 

　　　　　　Incorrect key file for table: '...'. Try to repair it

 　　　　2 、查询不能在表中找到行或返回不完全的数据。

　　　　 3 、Error: Table 'p' is marked as crashed and should be repaired 。

　　　　 4 、打开表失败： Can’t open file: ‘×××.MYI’ (errno: 145) 。

  ER_NOT_FORM_FILE  Incorrect information in file:
  ER_NOT_KEYFILE    Incorrect key file for table 'TTT'; try to repair it
  ER_OLD_KEYFILE    Old key file for table 'TTT'; repair it!
  ER_CANT_OPEN_FILE Can't open file: 'TTT' (errno: %d - %s)
 

　　3.预防 MySQL 表损坏

 　　可以采用以下手段预防mysql 表损坏： 

　　　　1 、定期使用myisamchk 检查MyISAM 表（注意要关闭mysqld ），推荐使用check table 来检查表（不用关闭mysqld ）。

　　　　2 、在做过大量的更新或删除操作后，推荐使用OPTIMIZE TABLE 来优化表，这样既减少了文件碎片，又减少了表损坏的概率。

　　　　3 、关闭服务器前，先关闭mysqld （正常关闭服务，不要使用kill -9 来杀进程）。

　　　　4 、使用ups 电源，避免出现突然断电的情况。

　　　　5 、使用最新的稳定发布版mysql ，减少mysql 本身的bug 导致表损坏。

　　　　6 、对于InnoDB 引擎，你可以使用innodb_tablespace_monitor来检查表空间文件内文件空间管理的完整性。

　　　　7 、对磁盘做raid ，减少磁盘出错并提高性能。

　　　　8 、数据库服务器最好只跑mysqld 和必要的其他服务，不要跑其他业务服务，这样减少死机导致表损坏的可能。

　　　　9 、不怕万一，只怕意外，平时做好备份是预防表损坏的有效手段。

　　4. MySQL 表损坏的修复

　　MyISAM 表可以采用以下步骤进行修复 ：

　　　　1、  使用 reapair table 或myisamchk 来修复。 

　　　　2、  如果上面的方法修复无效，采用备份恢复表。



　　具体可以参考如下做法：

　　阶段1 ：检查你的表

　　　　如果你有很多时间，运行myisamchk *.MYI 或myisamchk -e *.MYI 。使用-s （沉默）选项禁止不必要的信息。 

　　　　如果mysqld 服务器处于宕机状态，应使用--update-state 选项来告诉myisamchk 将表标记为' 检查过的' 。

　　　　你必须只修复那些myisamchk 报告有错误的表。对这样的表，继续到阶段2 。

　　　　如果在检查时，你得到奇怪的错误( 例如out of memory 错误) ，或如果myisamchk 崩溃，到阶段3 。

　　阶段2 ：简单安全的修复

　　　　注释：如果想更快地进行修复，当运行myisamchk 时，你应将sort_buffer_size 和Key_buffer_size 变量的值设置为可用内存的大约25% 。

　　　　首先，试试myisamchk -r -q tbl_name(-r -q 意味着“ 快速恢复模式”) 。这将试图不接触数据文件来修复索引文件。如果数据文件包含它应有的一切内容和指向数据文件内正确地点的删除连接，这应该管用并且表可被修复。开始修复下一张表。否则，执行下列过程：

　　　　在继续前对数据文件进行备份。

　　　　使用myisamchk -r tbl_name(-r 意味着“ 恢复模式”) 。这将从数据文件中删除不正确的记录和已被删除的记录并重建索引文件。

　　　　如果前面的步骤失败，使用myisamchk --safe-recover tbl_name 。安全恢复模式使用一个老的恢复方法，处理常规恢复模式不行的少数情况( 但是更慢) 。

　　　　如果在修复时，你得到奇怪的错误( 例如out of memory 错误) ，或如果myisamchk 崩溃，到阶段3 。 

　　阶段3 ：困难的修复

　　　　只有在索引文件的第一个16K 块被破坏，或包含不正确的信息，或如果索引文件丢失，你才应该到这个阶段。在这种情况下，需要创建一个新的索引文件。按如下步骤操做：

　　　　把数据文件移到安全的地方。

　　　　使用表描述文件创建新的( 空) 数据文件和索引文件：

　　　　shell> mysql db_name

　　　　mysql> SET AUTOCOMMIT=1;

　　　　mysql> TRUNCATE TABLE tbl_name;

　　　　mysql> quit 

　　　　如果你的MySQL 版本没有TRUNCATE TABLE ，则使用DELETE FROM tbl_name 。

　　　　将老的数据文件拷贝到新创建的数据文件之中。（不要只是将老文件移回新文件之中；你要保留一个副本以防某些东西出错。）

　　　　回到阶段2 。现在myisamchk -r -q 应该工作了。（这不应该是一个无限循环）。

　　　　你还可以使用REPAIR TABLE tbl_name USE_FRM ，将自动执行整个程序。

　　阶段4 ：非常困难的修复

　　　　只有.frm 描述文件也破坏了，你才应该到达这个阶段。这应该从未发生过，因为在表被创建以后，描述文件就不再改变了。

 　　　　从一个备份恢复描述文件然后回到阶段3 。你也可以恢复索引文件然后回到阶段2 。对后者，你应该用myisamchk -r 启动。

　　　　如果你没有进行备份但是确切地知道表是怎样创建的，在另一个数据库中创建表的一个拷贝。删除新的数据文件，然后从其他数据库将描述文件和索引文件移到破坏的数据库中。这样提供了新的描述和索引文件，但是让.MYD 数据文件独自留下来了。回到阶段2 并且尝试重建索引文件。
Log Sequence Number, LSN

Sharp Checkpoint 是一次性将 buffer pool 中的所有脏页都刷新到磁盘的数据文件，同时会保存最后一个提交的事务LSN。


fuzzy checkpoint就更加复杂了，它是在固定的时间点发生，除非他已经将所有的页信息刷新到了磁盘，或者是刚发生过一次sharp checkpoint，fuzzy checkpoint发生的时候会记录两次LSN，也就是检查点发生的时间和检查点结束的时间。但是呢，被刷新的页在并不一定在某一个时间点是一致的，这也就是它为什么叫fuzzy的原因。较早刷入磁盘的数据可能已经修改了，较晚刷新的数据可能会有一个比前面LSN更新更小的一个LSN。fuzzy checkpoint在某种意义上可以理解为fuzzy checkpoint从redo  log的第一个LSN执行到最后一个LSN。恢复以后的话，REDO LOG就会从最后一个检查点开始时候记录的LSN开始。

一般情况下大家可能fuzzy checkpoint的发生频率会远高于sharp checkpoint发生的频率，这个事毫无疑问的。不过当数据库关闭，切换redo日志文件的时候是会触发sharp checkpoint，一般情况是fuzzy checkpoint发生的更多一些。

一般情况下，执行普通操作的时候将不会发生检查点的操作，但是，fuzzy checkpoint却要根据时间推进而不停的发生。刷新脏页已经成为了数据库的一个普通的日常操作。

INNODB维护了一个大的缓冲区，以保证被修改的数据不会被立即写入磁盘。她会将这些修改过的数据先保留在buffer pool当中，这样在这些数据被写入磁盘以前可能会经过多次的修改，我们称之为写结合。这些数据页在buffer pool当中都是按照list来管理的，free list会记录那些空间是可用的，LRU list记录了那些数据页是最近被访问到的。flush list则记录了在LSN顺序当中的所有的dirty page信息，最近最少修改信息。

这里着重看一下flush list，我们知道innodb的缓存空间是有限的。如果buffer pool空间使用完毕，再次读取新数据就会发生磁盘读，也就是会发生flush操作，所以说就要释放一部分没有被使用的空间来保证buffer pool的可用性。由于这样的操作是很耗时的，所以说INNODB是会连续按照时间点去执行刷新操作，这样就保证了又足够的clean page来作为交换，而不必发生flush操作。每一次刷新都会将flush list的最老的信息驱逐，这样才能够保证数据库缓冲命中率是很高的一个值。这些老数据的选取是根据他们在磁盘的位置和LSN（最后一次修改的）号来确认数据新旧。

MySQL数据的日志都是混合循环使用的，但是如果这些事物记录的页信息还没有被刷新到磁盘当中的话是绝对不会被覆盖写入的。如果还没被刷新入磁盘的数据被覆盖了日志文件，那数据库宕机的话岂不是所有被覆盖写入的事物对应的数据都要丢失了呢。因此，数据修改也是有时间限制的，因为新的事物或者正在执行的事物也是需要日志空间的。日志越大，限制就越小。而且每次fuzzy checkpoint都会将最老最不被访问的数据驱逐出去，这也保证了每次驱逐的都是最老的数据，在下次日志被覆盖写入的时候都是已经被刷盘的数据的日志信息。最后一个老的,不被访问的数据的事物的LSN就是事务日志的 low-water标记，INNODB一直想提高这个LSN的值以保证buffer pool又足够的空间刷入新的数据，同时保证了数据库事务日志文件可以被覆盖写入的时候有足够的空间使用。将事务日志设置的大一些能够降低释放日志空间的紧迫性，从而可以大大的提高性能。

当innodb刷新 dirty page落盘的时候，他会找到最老的dirty  page对应的LSN并且将其标记为low-water，然后将这些信息记录到事物日志的头部，因此，每次刷新脏页都是要从flush  list的头部进行刷新的。在推进最老的LSN的标记位置的时候，本质上就是做了一次检查点。

当INNODB宕机的时候，他还要做一些额外的操作，第一：停止所有的数据更新等操作，第二：将dirty page in  buffer 的数据刷新落盘，第三：记录最后的LSN，因为我们上面也说到了，这次发生的是sharp checkpoint，并且，这个LSN会写入到没一个数据库文件的头部，以此来标记最后发生检查点的时候的LSN位置。

我们知道，刷新脏页数据的频率如果越高的话就代表整个数据库的负载很大，越小当然代表数据库的压力会小一点。将LOG 文件设置的很大能够再检查点发生期间减少磁盘的IO,总大小最好能够设置为和buffer pool大小相同，当然如果日志文件设置太大的话MySQL就会再crash recovery的时候花费更多的时间（5.5之前）。


http://mysqlmusings.blogspot.com/2011/04/crash-safe-replication.html

http://apprize.info/php/effective/6.html

http://mysqllover.com/?p=620

http://tech.uc.cn/?p=716

http://mysql.taobao.org/monthly/2016/05/01/

http://mysql.taobao.org/monthly/2015/06/01/

http://mysql.taobao.org/monthly/2015/05/01/

http://mysqllover.com/?p=376

http://hedengcheng.com/?p=183

https://gold.xitu.io/entry/5841225561ff4b00587ec651

https://yq.aliyun.com/articles/64677?utm_campaign=wenzhang&utm_medium=article&utm_source=QQ-qun&utm_content=m_7935

http://www.sysdb.cn/index.php/2016/01/14/innodb-recovery/

http://hedengcheng.com/?p=88InnoDB

http://mysqllover.com/?p=696

http://mysqllover.com/?p=213

http://mysql.taobao.org/monthly/2016/02/03/

http://mysql.taobao.org/monthly/2016/08/07/

http://mysql.taobao.org/monthly/2015/12/01/

http://mysqllover.com/?p=834

http://mysqllover.com/?p=1087

http://www.xuchunyang.com/2016/01/13/deak_lock/

http://mysqllover.com/?p=1119

http://www.askmaclean.com/archives/mysql-recover-innodb.html

http://www.askmaclean.com/archives/mysql%e4%b8%ad%e6%81%a2%e5%a4%8d%e4%bf%ae%e5%a4%8dinnodb%e6%95%b0%e6%8d%ae%e5%ad%97%e5%85%b8.html

http://www.thinkphp.cn/code/430.html

http://www.cnblogs.com/liuhao/p/3714012.html

http://louisyang.blog.51cto.com/8381303/1360394

http://mysql.taobao.org/monthly/2015/05/01/

http://hamilton.duapp.com/detail?articleId=34

https://twindb.com/undrop-tool-for-innodb/

https://twindb.com/tag/stream_parser/

http://jishu.y5y.com.cn/aeolus_pu/article/details/60143284

http://hedengcheng.com/?p=148

read_view

http://www.cnblogs.com/chenpingzhao/p/5065316.html

http://kabike.iteye.com/blog/1820553

http://www.sysdb.cn/index.php/2016/01/14/innodb-recovery/

http://mysqllover.com/?p=696

http://imysql.com/2014/08/13/mysql-faq-howto-shutdown-mysqld-fulgraceful.shtml

http://11879724.blog.51cto.com/11869724/1872928

http://coolnull.com/3145.html

http://mysqllover.com/?p=594

http://mysqllover.com/?p=87

http://mysqllover.com/?p=581

http://keithlan.github.io/2016/06/23/gtid/

http://mysqlmusings.blogspot.com/2011/04/crash-safe-replication.html

https://dbarobin.com/2015/08/29/mysql-optimization-under-ssd/

http://www.orczhou.com/index.php/2010/12/more-about-mysql-innodb-shutdown/

https://www.slideshare.net/frogd/inno-db-15344119

https://www.xaprb.com/blog/2011/01/29/how-innodb-performs-a-checkpoint/

http://www.cnblogs.com/chenpingzhao/p/5107480.html

https://github.com/zhaiwx1987/innodb_ebook/blob/master/innodb_adaptive_hash.md




隔离级别

详细可以查看row_search_for_mysql()中的实现，实际上也就是row_search_mvcc()函数的实现。

row_search_for_mysql()   
 |-row_search_no_mvcc()       # 对于MySQL内部使用的表(用户不可见)，不需要MVCC机制
 |-row_search_mvcc()

row_search_no_mvcc()用于MySQL的内部表使用，通常是一些作为一个较大任务的中间结果存储，所以希望其可以尽快处理，因此不需要MVCC机制。

事务的隔离级别在trx->isolation_level中定义，其取值也就是如下的宏定义。

#define TRX_ISO_READ_UNCOMMITTED        0
#define TRX_ISO_READ_COMMITTED          1
#define TRX_ISO_REPEATABLE_READ         2
#define TRX_ISO_SERIALIZABLE            3


在不同的隔离级别下，可见性的判断有很大的不同。

READ-UNCOMMITTED
在该隔离级别下会读到未提交事务所产生的数据更改，这意味着可以读到脏数据，实际上你可以从函数row_search_mvcc中发现，当从btree读到一条记录后，如果隔离级别设置成READ-UNCOMMITTED，根本不会去检查可见性或是查看老版本。这意味着，即使在同一条SQL中，也可能读到不一致的数据。

    READ-COMMITTED
    在该隔离级别下，可以在SQL级别做到一致性读，当事务中的SQL执行完成时，ReadView被立刻释放了，在执行下一条SQL时再重建ReadView。这意味着如果两次查询之间有别的事务提交了，是可以读到不一致的数据的。

    REPEATABLE-READ
    可重复读和READ-COMMITTED的不同之处在于，当第一次创建ReadView后（例如事务内执行的第一条SEELCT语句），这个视图就会一直维持到事务结束。也就是说，在事务执行期间的可见性判断不会发生变化，从而实现了事务内的可重复读。

    SERIALIZABLE
    序列化的隔离是最高等级的隔离级别，当一个事务在对某个表做记录变更操作时，另外一个查询操作就会被该操作堵塞住。同样的，如果某个只读事务开启并查询了某些记录，那么另外一个session对这些记录的更改操作是被堵塞的。内部的实现其实很简单：
        对InnoDB表级别加LOCK_IS锁，防止表结构变更操作
        对查询得到的记录加LOCK_S共享锁，这意味着在该隔离级别下，读操作不会互相阻塞。而数据变更操作通常会对记录加LOCK_X锁，和LOCK_S锁相冲突，InnoDB通过给查询加记录锁的方式来保证了序列化的隔离级别。

注意不同的隔离级别下，数据具有不同的隔离性，甚至事务锁的加锁策略也不尽相同，你需要根据自己实际的业务情况来进行选择。





SELECT count_star, sum_timer_wait, avg_timer_wait, event_name FROM events_waits_summary_global_by_event_name WHERE count_star > 0 AND event_name LIKE "wait/synch/%" ORDER BY sum_timer_wait DESC LIMIT 20;




最新的事务ID通过trx_sys_get_new_trx_id()函数获取，每次超过了TRX_SYS_TRX_ID_WRITE_MARGIN次数后，都会调用trx_sys_flush_max_trx_id()函数刷新磁盘。






innodb_force_recovery变量对应源码中的srv_force_recovery变量，
 


当innodb_fast_shutdown设置为0时，会导致purge一直工作近两个小时。？？？？？

从5.5版本开始，purge任务从主线程中独立出来；5.6开始支持多个purge线程，可以通过innodb_purge_threads变量控制。

purge后台线程的最大数量可以有32个，包括了一个coordinator线程，以及多个worker线程。

在innobase_start_or_create_for_mysql()函数中，会创建srv_purge_coordinator_thread以及srv_worker_thread线程。


srv_purge_coordinator_thread()
 |-srv_purge_coordinator_suspend()   如果不需要purge或者上次purge记录数为0，则暂停
 |-srv_purge_should_exit()           判断是否需要退出；fast_shutdown=0则等待所有purge操作完成
 |-srv_do_purge()                    协调线程的主要工作，真正调用执行purge操作的函数
 |
 |-trx_purge()                       防止上次循环结束后又新的记录写入，此处不再使用worker线程
 |
 |-trx_purge()                       最后对history-list做一次清理，确保所有worker退出

srv_worker_thread()


最后一次做trx_purge()时，为了防止执行时间过程，批量操作时不再采用innodb_purge_batch_size(300)指定的值，而是采用20。


InnoDB的数据组织方式采用聚簇索引，也就是索引组织表，而二级索引采用(索引键值,主键键值)组合来唯一确定一条记录。
无论是聚簇索引，还是二级索引，每条记录都包含了一个DELETED-BIT位，用于标识该记录是否是删除记录；除此之外，聚簇索引还有两个系统列：DATA_TRX_ID，DATA_ROLL_PTR，分别表示产生当前记录项的事务ID以及指向当前记录的undo信息。



从聚簇索引行结构，与二级索引行结构可以看出，聚簇索引中包含版本信息(事务号+回滚指针)，二级索引不包含版本信息，二级索引项的可见性如何判断？？？？


InnoDB存储引擎在开始一个RR读之前，会创建一个Read View。Read View用于判断一条记录的可见性。Read View定义在read0read.h文件中，其中最主要的与可见性相关的属性如下：

class ReadView {
private:
  trx_id_t        m_low_limit_id;  //
};


ReadView::prepare()


copy_trx_ids
mtr_commit(struct mtr_t*)                 提交一个mini-transaction，调用mtr_t::commit()
 |-mtr_t::Command::execute()              写redo-log，将脏页添加到flush-list，并释放占用资源
   |-mtr_t::Command::prepare_write()      准备写入日志
   | |-fil_names_write_if_was_clean()
   |-mtr_t::Command::finish_write()   


测试场景#1 Drop Database (innodb_file_per_table=ON)
OFF代表MySQL是共享表空间，也就是所有库的数据都存放在一个ibdate1文件中；ON代表每个表的存储空间都是独立的。

ibd是MySQL数据文件、索引文件，二进制文件无法直接读取；frm是表结构文件，可以直接打开。如果innodb_file_per_table 无论是ON还是OFF，都会有这2个文件，区别只是innodb_file_per_table为ON的时候，数据时放在 .idb中，如果为OFF则放在ibdata1中。






 

## Buffer Pool

https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html


BP 通过 Least Recently Used, LRU 算法组织，当需要插入新数据时，将最近最少使用的页去除，并将新页插入到列表的中间，也就是 "midpoint insertion strategy" 。

头部的子链表，表示最近读取过；尾部的子链表表示最近很少读取；也就是说经常使用的页会放在头部，而需要去除的会放在尾部。

### 简介

单纯的 LRU 算法有一个缺点：如果有某一个查询做了一次全表扫描，如备份、DDL 等，都可能会导致整个 LRU 链表中的数据块都被替换了，甚至很多热点数据也会被替换，而这些新进的数据块可能在这一次查询之后就再也不会被读到了；此时也就是说 BP 被污染了。

即使采用上述的 midpoint 方法，也就是说当数据块需要从数据文件中读取时 (也包括了预读)，首先会放到 old sublist 的头部
(midpoint)。然后，如果有对这个数据块的访问，那么就将这个数据块放到 new sublist 的首部。

一般来说，一个数据块被取出后，立刻会有读取，也就很快会被放到 new sublist 的头部。一种糟糕的情况是，如果是 mysqldump 访问全部数据块，也就会导致所有的数据块被放到 new sublist；这样 BP 也会被全部污染。

为了解决这个问题，可以设置 ```innodb_old_blocks_time(ms)``` 参数，当页被插入到 midpoint 后，必须要在 old sublist 的头部停留超过上述的时间后，才有可能被转移到 new sublist。

参数 ```innodb_old_blocks_time``` 可以动态设置，在执行一些全表扫描时，可以将上述参数设置为比较大的值，操作完成之后再恢复为 0 。

{% highlight text %}
SET GLOBAL innodb_old_blocks_time = 1000;
... perform queries that scan tables ...
SET GLOBAL innodb_old_blocks_time = 0;
{% endhighlight %}

还有一种情况时，希望数据加载的缓存中。例如，在进行性能测试时，通常会执行一次全表扫描，将磁盘中的数据加载到内存中，那么，此时就应该将上述的参数设置为 0 。


### 多实例配置

<!--Multiple Buffer Pool Instances-->

当服务器的内存比较大时，如果多个线程读取 BP 数据，会导致瓶颈；此时，可以将 BP 分为几个实例 (instance)，从而提高并发，减少资源冲突。每个实例管理自己的 free lists、flush lists、LRUs、互斥锁 (mutex) 以及其它与 BP 相关的结构。

可以通过 innodb_buffer_pool_instances 参数配置实例个数，默认为 1，最大可以配置为 64；只有当 innodb_buffer_pool_size 的值大于 1G 时才会生效，而且每个实例均分 BP 缓存。

通过多个 BP 可以减小竞争，每个页(page)将通过一个 hash 函数随机分配到 BP 中。????????

### 预读策略

预读其实是基于这样的判断，在读取磁盘数据时，接下来的请求，有很大可能要读取周围的数据；为此，InnoDB 会异步读取该页周围的数据，从而减小 IO 。

InnoDB 提供了顺序和随机两种策略。

#### 顺序预读

<!--
buf_read_ahead_linear()

也就是 linear read-ahead ，会根据
https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-read_ahead.html
-->




### 配置参数

* innodb_buffer_pool_size<br>BP 缓存大小，包括了数据以及索引，对于专用的数据库服务器上通常为物理内存的 70% ~ 80%；5.7.5 版本后可以动态调整，调整时按 Block 进行；
* innodb_buffer_pool_chunk_size<br>动态调整时 Block 的大小；
* innodb_buffer_pool_instances<br>BP 缓存的实例数，只有当 BP 大于 1G 才会生效；
* innodb_old_blocks_pct<br>old-sublist 的分割点，可以为 5~95，默认为 37 也就是 3/8；
* innodb_old_blocks_time (ms)<br>指定页必须在 old-sublist 中待多长时间之后，才有可能被移动到新列表中；


<!--
    innodb_flush_method: 这个控制Innodb的IO形为，什么:fsync, O_DSYNC之类的，这里不做过多介绍， 建议使用: O_DIRECT, 这样减少操作系统级别VFS的缓存使用内存过多和Innodb本身的buffer的缓存冲突，同时也算是给操作系统减少点压力。
    innodb_max_dirty_pages_pct : 这个参数据控制脏页的比例如果是innodb_plugin或是MySQL5.5以上的版本，建议这个参数可以设制到75%-90%都行。如果是大量写入，而且写入的数据不是太活跃，可以考虑把这个值设的低一点。 如果写入或是更新的数据也就是热数据就可以考虑把这个值设为：95%
    innodb_log_file_size : 这个可以配置256M以上，建议有两个以前的日志文件（innodb_log_files_in_group). 如果对系统非常大写的情况下，也可以考虑用这个参数提高一下性能，把文件设的大一点，减少checkpiont的发生。 最大可以设制成：innodb_log_files_in_group * innodb_log_file_size < 512G(percona, MySQL 5.6) 建议设制成: 256M -> innodb_buffer_pool_size/innodb_log_file_in_group 即可。
    innodb_log_buffer_size : 如果没在大事务，控制在8M-16M即可。

其它对IO有影响的参数(以5.6为准）

    innodb_adaptive_flushing 默认即可
    innodb_change_buffer_max_size 如果是日值类服务，可以考虑把这个增值调到 50
    innodb_change_buffering 默认即可
    innodb_flush_neighors 默认是开的， 这个一定要开着，充分利用顺序IO去写数据。
    innodb_lru_scan_depth: 默认即可 这个参数比较专业。
    innodb_max_purge_lag 默认没启用，如果写入和读取都量大，可以保证读取优先，可以考虑使用这个功能。
    innodb_random_read_ahead 默认没开启，属于一个比较活跃的参数，如果要用一定要多测试一下。 对用passport类应用可以考虑使用
    innodb_read_ahead_threshold 默认开启：56 预读机制可以根据业务处理，如果是passprot可以考虑关闭。如果使用innodb_random_read_ahead,建议关闭这个功能
    sync_binlog 默认即可： 0
    innodb_rollback_segments 默认即可: 128
另外5.6的undo log也可以独立配置了，建议单独配置出来。
-->
    innodb_read_io_threads 默认为：4 可以考虑8
    innodb_write_io_threads 默认为：4 可以考虑8

### 源码解析

Buffer Pool 的初始化入口在 ```innobase_start_or_create_for_mysql()``` 函数中，调用流程如下。

{% highlight text %}
buf_pool_init()                     # 初始化buffer pool主函数入口
 |-buf_pool_init_instance()         # 1.1 每个buf-pool大小为srv_buf_pool_size/instance，通过该函数初始化各个实例
 | |-UT_LIST_INIT(buf_pool->free)   # 2.1 初始化buf pool的free list
 | |
 | |-buf_chunk_init()               # 初始化buffer pool中的chunk
 | | |-ut_2pow_round()              # 2.2 计算每个buf pool实际所需空间，空间必须按照page_size对齐
 | | |-ut_2pow_round()              #     必须为每个page分配一个内存block结构，用于管理内存page
 | | |-os_mem_alloc_large()         # 2.3 为buf pool分配大块内存，对于不同系统配置，调用不同函数
 | | | |-shmget() shmat() shmct()   #     若用了大页(Huge Page)，则使用这些方法分配空间
 | | | |-VirtualAlloc()             #     Win平台使用该函数分配空间
 | | | |-ut_malloc_low()            #     若未使用MMAP，则调用该函数分配空间
 | | | |-mmap()                     #     若使用MMAP，则调用mmap函数分配空间
 | | |-ut_align()                   # 2.4 将分配出来的mem空间，按page size对齐，作为page的起点
 | | |-... ...                      # 2.5 将mem划分为两部分：前部分作为block结构空间；后部分作为page空间
 | | |                              #     page空间每一个page的起始位置，必须按照page size对齐
 | | |                              #     block结构起始位置为mem空间的0号位置
 | | |                              #     page空间的起始位置，为预留足够的block结构之后的第一个frame位置
 | | |-buf_block_init()             # 2.6 为每个page指定一个block头结构，并初始化各种mutex与lock
 | | |-UT_LIST_ADD_LAST()           # 2.7 将page加入buf pool的free list链表，等待分配
 | |-hash_create()                  # 2.8 创建相应的hash表，若page对应于文件中的一个页，则在page hash表中存在
 |                                  #     便于page在内存中的快速定位
 |
 |-buf_pool_set_sizes()             # 1.2 设置大小参数
 |-buf_LRU_old_ratio_update()       # 1.3 设置buf_pool.LRU中oldList部分所占的比率，默认为3/8
 |-btr_search_sys_create()
{% endhighlight %}

其中比较重要的参数是 innodb_old_blocks_pct，该参数运行时可以调整，初始值为 3/8 。

若设置不同的 old ratio，则涉及到调整 buf_pool-&gt;LRU_old 指向的位置，LRU_old 指向的是 LRU list 中位于 old ratio 处的 block 位置，也就是说 old ration 调整，LRU_old 需要相应的做出调整。



### 页面管理

在 MySQL 5.5 之后，InnoDB 支持多 Buffer Pool Instance，内存中有多个 buffer pool 管理，如果此时指定一个 page，需要确定属于哪个 buffer pool 的。

其入参是给定一个 page 的 tablespace id 与 offset，然后可以定位到对应的管理 buffer pool 。

buf_pool_t* buf_pool_get(space_id, offset)    @ include/buf0buf.ic
{
    ignored_offset = offset >> 6;                         // 将page_no右移6位
    fold = buf_page_address_fold(space, ignored_offset);  // 计算fold
    index = fold % srv_buf_pool_instances;                // 并从fold计算出对应的buffer pool index
    return(&buf_pool_ptr[index]);
}

因为目前 InnoDB 的一个 read ahead 是 64 个 page，因此右移 6 位能够保证一个 read ahead area 中的页，都属于同一个 buffer pool 管理。





## Page Cleaner

为了提升刷脏效率，在 MySQL 5.7.4 版本里引入了多个 page cleaner 线程，从而可以达到并行刷脏的效果。采用协调线程+工作线程模式，协调线程本身也是工作线程，可通过 ```innodb_page_cleaners``` 变量设置，例如设置为 8 时就是一个协调线程，加 7 个工作线程。

{% highlight text %}
buf_flush_page_cleaner_coordinator

buf_flush_page_cleaner_worker
 |-my_thread_init()
 |-###BEGIN while
 |-os_event_wait()             等待page_cleaner->is_requested事件
 |-pc_flush_slot()
{% endhighlight %}

在启动时会初始化一个slot数组，大小为buffer pool instance的个数(buf_flush_page_cleaner_init)。

协调线程在决定了需要flush的page数和lsn_limit后，会设置slot数组，将其中每个slot的状态设置为PAGE_CLEANER_STATE_REQUESTED, 并设置目标page数及lsn_limit，然后唤醒worker线程 (pc_request)

worker线程被唤醒后，从slot数组中取一个未被占用的slot，修改其状态，表示已被调度，然后对该slot所对应的buffer pool instance进行操作。

为了支持对单个bp instance进行LRU/FLUSH_LIST的刷新，对原有代码做了大量的改动，worker线程可以直接调用buf_flush_LRU_list 及buf_flush_do_batch 指定buffer pool进行flush操作。 互相之间不干扰，因此可以并行刷脏。 改动整体而言比较简单。







## 简介

通常有几种方式关闭 MySQL 服务器，常见有如下：

* 通过 ```mysqladmin shutdown``` 命令关闭服务，会调用my_xxxx函数，服务端通过COM_SHUTDOWN处理，会新建一个单独线程处理；
* 向服务器发送 SIGTERM 信号，该信号会在my_init_signals()初始化；

线程调用函数为kill_server_thread

{% highlight text %}
main()
 |-start_signal_handler()
   |-mysql_thread_create()                     ← 创建signal_hand()单独线程

signal_hand()
 |-pthread_kill()

dispatch_command()
 |-shutdown()                                 ← COM_SHUTDOWN:
 | |-general_log_print()                      ← Got shutdown command for level
 | |-kill_mysql()
 |   |-pthread_kill()                         ← 发送SIGTERM信号
 |
 |-my_thread_join()                           ← 等待上述的signal_hand()处理线程
 |-clean_up()
   |-plugin_shutdown()
     |-plugin_deinitialize()
       |-ha_finalize_handlerton()
         |-hton->panic()                      ← 调用插件的关闭函数，innobase_end

innobase_end()
 |-srv_fast_shutdown                          ← 根据参数innodb_fast_shutdown判断是否快速关闭
 |-hash_table_free()                          ← 释放innodb表占用的内存
 |-innobase_shutdown_for_mysql()
 | |-fts_optimize_shutdown()
 | |-dict_stats_shutdown()
 | |-logs_empty_and_mark_files_at_shutdown()  ← 1. 将buffer pool落盘，并将LSN写入表空间，主要函数，后面均为资源清理
 |   |-ib::info()                                  打印Starting shutdown日志
 |   |-srt_shutdown_state                          进入SRV_SHUTDOWN_CLEANUP状态
 | |-srv_conc_get_active_threads()            ←    正常应该无线程，有则打印到日志
 | |
 | |-srv_shutdown_all_bg_threads()            ← 2. 关闭InnoDB创建的后台线程
 | |-fclose()                                 ←    关闭InnoDB打开的文件
 | |-dict_stats_thread_deinit()
 | |
 | |-                                         ← 3. 释放mutexes
 | |
 | |-                                         ← 4. 释放内存
 | |
 | |-os_thread_free()                         ← 5. 释放线程相关的资源
 | |
 | |-sync_check_close()                       ← 6. 释放同步相关资源
 |
 |-innobase_space_shutdown()
{% endhighlight %}




写的比较简略//////
———————————

我们通过mysqladmin来进行shutdown，会对mysqld发送SIGKILL信号，当接收到信号后，mysqld创建一个新的线程，线程调用函数为kill_server_thread
另外还可以通过调用COM_SHUTDOWN来关闭mysqld，没尝试过。当然最终调用的函数kill_server_thread函数。
kill_server_thread->kill_server
设置kill_in_progress=true，防止重复关闭mysqld，每个新的kill线程都要先判断这个值
设置abort_loop=1，这可以让某些循环等待的线程退出循环
调用close_connections
a.关闭所有线程连接
b.关闭slave（end_slave()）
调用unireg_end->clean_up，主要做以下事情
a.清理一些全局结构体及内存资源
b.依次关闭插件plugin_shutdown




释放一些全局内存和锁资源

innobase_shutdown_for_mysql流程：
a.logs_empty_and_mark_files_at_shutdown
    这个函数可以看做是innodb shutdown的主要函数。
    |–>首先打印一条信息(InnoDB: Starting shutdown)， 也就是我们在alter log里看到的，表示innodb shutdown开始 了。
    srv_shutdown_state = SRV_SHUTDOWN_CLEANUP;
    |–>当

        1.error monitor线程、lock_timeout线程（用于唤醒其他等待锁的线程）、以及monitor线程处于活跃状态时，loop

        2.当前分配的事务数大于0(trx_n_mysql_transactions>0)或者存在状态不为Prepare的事务(UT_LIST_GET_LEN(trx_sys->trx_list) > trx_n_prepared), loop

        3. 存在任意一个活跃后台线程时， loop

        4.log_sys->n_pending_checkpoint_writes>0 或者log_sys->n_pending_writes大于0时，loop

        5. 存在pending io时(buf_pool_check_no_pending_io) ，  loop

    |–>当UNIV_LOG_ARCHIVE被定义时，调用log_archive_all()写归档日志，log archive 貌似已被弃用，但代码还在。
    |–>当srv_fast_shutdown=2时

        log_buffer_flush_to_disk(); 将缓冲的innodb 日志flush到磁盘

        检查如果有活跃后台线程，goto loop

        关闭所有innodb文件，调用函数fil_close_all_files，这里并不需要确保没有pending的I/O或者未刷到文件的脏数据
        srv_shutdown_state = SRV_SHUTDOWN_LAST_PHASE;

        return到上层函数

    |–>调用函数log_make_checkpoint_at(IB_ULONGLONG_MAX, TRUE)使用最近的lsn做checkpoint。
    |–>将可能被OS缓存的表空间文件和日志文件刷到磁盘上

    fil_flush_file_spaces(FIL_TABLESPACE);

    fil_flush_file_spaces(FIL_LOG);
    srv_shutdown_state = SRV_SHUTDOWN_LAST_PHASE;
    |–>fil_write_flushed_lsn_to_data_files && fil_flush_file_spaces(FIL_TABLESPACE);， 将LSN写到每个ibd文件中
    |–>fil_close_all_files(); 关闭所有innodb文件

srv_shutdown_state = SRV_SHUTDOWN_EXIT_THREADS;
b.退出所有innodb后台线程，包括master线程、purge线程以及其他io线程
当srv_fast_shutdown= 0时， 会做完全的purge 和insert buffer merge
这些在对应的线程函数里会做判断
c.btr_search_disable(); //禁止adaptive hash index并清空index
d.释放insert buffer、log、锁、事务、adpative hash index、文件、buffer pool等子系统的内存和其他资源
总的来说，0 比1 多做了insert buffer merge 和full purge，1比2多做了checkpoint和刷文件落地到磁盘的过程，2只做了将日志刷到磁盘





## 参考

[Reference Manual - The Server Shutdown Process](https://dev.mysql.com/doc/refman/5.7/en/server-shutdown.html)





















#######redo-log文件
redo-log保存在innodb_log_group_home_dir参数指定的目录下，文件名为ib_logfile*；undo保存在共享表空间ibdata*文件中。

InnoDB的redo log可控制文件大小以及文件个数，分别通过innodb_log_file_size和innodb_log_files_in_group控制，总大小为两者之积。日志顺序写入，而且文件循环使用。

简单来说，InnoDB中的两个核心参数innodb_buffer_pool_size、innodb_log_file_size，分别定义了数据缓存和redo-log的大小，而后者的大小也决定了可以允许buffer中可以有多少脏页。当然，也不能因此就增大redo-log文件的大小，如果这样，可能会导致系统启动时Crash Recovery时间增大。




LSN对应了日志文件的偏移量，为了减小故障恢复时间，引入了Checkpoint机制，

InnoDB在启动时会自动检测InnoDB数据和事务日志是否一致，是否需要执行相应的操作？？？保证数据一致性；当然，故障恢复时间与事务日志的大小相关。


checkpoint会将最近写入的LSN



## CheckPoint

{% highlight text %}
mysql> SHOW ENGINE INNODB STATUS\G
---
LOG
---
Log sequence number 293590838           LSN1事务创建时一条日志
Log flushed up to   293590838
Pages flushed up to 293590838
Last checkpoint at  293590829
0 pending log flushes, 0 pending chkp writes
1139 log i/o's done, 0.00 log i/o's/second
{% endhighlight %}

如上的信息是在 log_print() 函数中打印。

{% highlight cpp %}
void log_print( FILE* file)
{
    ... ...
    fprintf(file,
        "Log sequence number " LSN_PF "\n"
        "Log flushed up to   " LSN_PF "\n"
        "Pages flushed up to " LSN_PF "\n"
        "Last checkpoint at  " LSN_PF "\n",
        log_sys->lsn,
        log_sys->flushed_to_disk_lsn,
        log_buf_pool_get_oldest_modification(),
        log_sys->last_checkpoint_lsn);
    ... ...
}
{% endhighlight %}

通常有两种 Checkpoint，分别为：Sharp Checkpoint、Fuzzy Checkpoint；前者在正常关闭数据库时使用，会将所有脏页刷回磁盘；后者，会在运行时使用，用于部分脏页的刷新。

其中，后者的部分脏页刷新机制主要有以下几种：

Master Thread Checkpoint
主线程会以每秒或每十秒的速度从缓冲池的脏页列表中将一定比例的脏页刷回到磁盘；这个过程是异步的，不会阻塞查询线程。

FLUSH_LRU_LIST Checkpoint
InnoDB要保证LRU列表中有足够的空闲页可以使用，如果没有，会将LRU列表尾端的页淘汰，如果被淘汰的页中有脏页，会强制执行Checkpoint刷回脏页数据到磁盘，显然这会阻塞用户查询线程。从InnoDB1.2.X版本开始，这个检查放到单独的Page Cleaner Thread中进行，并且用户可以通过innodb_lru_scan_depth控制LRU列表中可用页的数量，默认值为 1024。

Async/Sync Flush Checkpoint？？？？？？
是指重做日志文件不可用时，需要强制将脏页列表中的一些页刷新回磁盘。这可以保证重做日志文件可循环使用。在InnoDB1.2.X版本之前，Async Flush Checkpoint会阻塞发现问题的用户查询线程，Sync Flush Checkpoint会阻塞所有查询线程。InnoDB1.2.X之后放到单独的Page Cleaner Thread。
https://www.percona.com/blog/2011/04/04/innodb-flushing-theory-and-solutions/

Dirty Page too much Checkpoint
脏页数量太多时，InnoDB引擎会强制进行Checkpoint；其目的还是为了保证缓冲池中有足够可用的空闲页，可以通过参数innodb_max_dirty_pages_pct来设置。


{% highlight text %}
log_make_checkpoint_at()
 |-log_preflush_pool_modified_pages()
 |-log_checkpoint()                         并不从BP中刷脏页，只检查BP中的最大LSN，然后刷新到磁盘
   |-log_mutex_enter()                      持有log_sys->mutex锁
   |-log_buf_pool_get_oldest_modification()
   | |-buf_pool_get_oldest_modification()   遍厉所有BP实例，获取最大lsn，之前都已经写入磁盘
   |
   |-fil_names_clear()
   | |-mtr_t::commit_checkpoint()
   |
   |-log_write_up_to()
   |
   |-log_write_checkpoint_info()
     |-log_group_checkpoint()               将checkpoint信息写入redolog头部，两个写入点轮流写入


buf_flush_page_cleaner_coordinator()
 |-buf_flush_page_cleaner_coordinator()
   |-page_cleaner_flush_pages_recommendation()
     |-af_get_pct_for_dirty()
       |-buf_get_modified_ratio_pct()
         |-buf_get_total_list_len()

buf_flush_page_cleaner_worker()
{% endhighlight %}

checkpoint 信息分别保存在 ib_logfile0 的 512 字节和 1536(3*512) 字节处，每个 checkpoint 默认大小为 512 字节，InnoDB 的 checkpoint 主要有 3 部分信息组成：

* checkpoint no<br>每次写入都会递增，用于轮流写入 redo log 的头部的两部分，可以通过该值判断那个比较新；
* checkpoint lsn<br>记录了产生该 checkpoint 时 log_sys->next_checkpoint_lsn 是flush的LSN，确保在该LSN前面的数据页都已经落盘，不再需要通过redo log进行恢复

checkpoint offset

checkpoint offset主要记录了该checkpoint产生时，redo log在ib_logfile中的偏移量，通过该offset位置就可以找到需要恢复的redo log开始位置。


checkpoint会将最近写入的LSN

checkpoint 如何保证原子操作？？？？


读取redo然后保存在hash表中，redo会不会有顺序？此时已经离散，会不会有问题？？？







####### 崩溃恢复(Crash Recovery)
Crash Recovery的起点，Checkpoint LSN存储位置？
InnoDB如何完成Redo日志的重做？
InnoDB如何定位哪些事务需要Rollback？
Crash Recovery需要等待Rollbach完成吗？
InnoDB各版本，在Crash Recovery流程上做了哪些优化？
mysqld_safe是否存在自动重启功能？

ha_recover

{% highlight text %}
innobase_init()
 |-innobase_start_or_create_for_mysql()
   |
   |-recv_sys_create()   创建崩溃恢复所需要的内存对象
   |-recv_sys_init()
   |
   |-srv_sys_space.open_or_create()                 通过系统表空间，获取flushed_lsn
   | |-read_lsn_and_check_flags()
   |   |-open_or_create()                           打开系统表空间
   |   |-read_first_page()                          读取第一个page
   |   |-buf_dblwr_init_or_load_pages()             加载double write buffer，如果ibdata日志损坏，则通过dblwr恢复
   |   |-validate_first_page()                      校验第一个页是否正常
   |   |-restore_from_doublewrite()                 如果有异常，则从dblwr恢复
   |
   |-srv_undo_tablespaces_init()                    对于undo log表空间恢复
   |
   |-recv_recovery_from_checkpoint_start()          ***从redo-log的checkpoint开始恢复；注意，正常启动也会调用
   | |-buf_flush_init_flush_rbt()                   创建一个红黑树，用于加速插入flush list
   | |-recv_recovery_on=true                        表示崩溃恢复已经开始，很多代码逻辑会通过该变量进行判断
   | |-recv_find_max_checkpoint()                   从日志中，找出最大的偏移量
   | |-
   | |-recv_group_scan_log_recs()
   | | |-recv_scan_log_recs()
   | |-recv_group_scan_log_recs()
   | |-recv_group_scan_log_recs()
   | |-recv_init_crash_recovery_spaces()
   |
   |-trx_sys_init_at_db_start()                     完成undo部分操作：收集未成功提交事务，按类别划分，前期准备
   | |-trx_sysf_get()                               完成undo-log的收集
   | |-trx_lists_init_at_db_start()                 根据undo信息重建未提交事务
   |
   |-recv_apply_hashed_log_recs()                   应用redo日志
   | |-recv_recover_page()                          实际调用recv_recover_page_func()
   |
   |-trx_purge_sys_create()                         构建purge，至此undo信息和undo事务创建结束
   |
   |-recv_recovery_from_checkpoint_finish()         完成崩溃恢复


   |-os_thread_create() 创建srv_master_thread线程
{% endhighlight %}


主线程主要完成 purge、checkpoint、dirty pages flush 等操作。



Database was not shutdown normally!   # InnoDB开始Crash Recovery{recv_init_crash_recovery_spaces()}
Starting crash recovery.



1. 读取Checkpoint LSN
2. 从Checkpoint LSN开始向前遍历Redo Log File
   重做从Checkpoint LSN开始的所有Redo日志
3. 重新构造系统崩溃时的事务
   Commit事务，等待Purge线程回收
   Prepare事务，由MySQL Server控制提交或者回滚(与Binlog 2PC相关)
   Active事务，回滚
4. 新建各种后台线程，Crash Recovery完成返回


正常关闭时，会在flush redo log和脏页后，做一次完全同步的checkpoint，并将checkpoint的LSN写到第一个ibdata文件的第一个page中，详细可以参考fil_write_flushed_lsn()。




innobase_start_or_create_for_mysql()
 |-log_group_init()
   |-log_calc_max_ages()


        log_sys->log_group_capacity = smallest_capacity;

        log_sys->max_modified_age_async = margin
                - margin / LOG_POOL_PREFLUSH_RATIO_ASYNC;
        log_sys->max_modified_age_sync = margin
                - margin / LOG_POOL_PREFLUSH_RATIO_SYNC;

        log_sys->max_checkpoint_age_async = margin - margin
                / LOG_POOL_CHECKPOINT_RATIO_ASYNC;
        log_sys->max_checkpoint_age = margin;




http://mysqllover.com/?p=376

http://hedengcheng.com/?p=183

http://mysql.taobao.org/monthly/2015/05/01/

http://mysql.taobao.org/monthly/2016/05/01/

http://tech.uc.cn/?p=716

http://hedengcheng.com/?p=88InnoDB

http://mysqllover.com/?p=620

http://apprize.info/php/effective/6.html

http://www.cnblogs.com/chenpingzhao/p/5107480.html

https://www.xaprb.com/blog/2011/01/29/how-innodb-performs-a-checkpoint/

数据库内核分享

https://www.slideshare.net/frogd/inno-db-15344119

检查保存到磁盘的最大checkpoint LSN与redo-log的LSN是否一致；


MySQL · 引擎特性 · InnoDB 崩溃恢复过程

http://mysql.taobao.org/monthly/2015/06/01/













https://blogs.oracle.com/mysqlinnodb/entry/repeatable_read_isolation_level_in



http://mysql.taobao.org/monthly/2015/12/01/
http://hedengcheng.com/?p=148
read_view
http://www.cnblogs.com/chenpingzhao/p/5065316.html
http://kabike.iteye.com/blog/1820553
http://www.sysdb.cn/index.php/2016/01/14/innodb-recovery/
http://mysqllover.com/?p=696

隔离级别
详细可以查看row_search_mvcc()中的实现


row_search_for_mysql()
 |-row_search_no_mvcc()       # 对于MySQL内部使用的表(用户不可见)，不需要MVCC机制
 |-row_search_mvcc()

row_search_no_mvcc()用于MySQL的内部表使用，通常是一些作为一个较大任务的中间结果存储，所以希望其可以尽快处理，因此不需要MVCC机制。


innodb_force_recovery变量对应源码中的srv_force_recovery变量，




当innodb_fast_shutdown设置为0时，会导致purge一直工作近两个小时。？？？？？

从5.5版本开始，purge任务从主线程中独立出来；5.6开始支持多个purge线程，可以通过innodb_purge_threads变量控制。

purge后台线程的最大数量可以有32个，包括了一个coordinator线程，以及多个worker线程。

在innobase_start_or_create_for_mysql()函数中，会创建srv_purge_coordinator_thread以及srv_worker_thread线程。


srv_purge_coordinator_thread()
 |-srv_purge_coordinator_suspend()   如果不需要purge或者上次purge记录数为0，则暂停
 |-srv_purge_should_exit()           判断是否需要退出；fast_shutdown=0则等待所有purge操作完成
 |-srv_do_purge()                    协调线程的主要工作，真正调用执行purge操作的函数
 |
 |-trx_purge()                       防止上次循环结束后又新的记录写入，此处不再使用worker线程
 |
 |-trx_purge()                       最后对history-list做一次清理，确保所有worker退出

srv_worker_thread()


最后一次做trx_purge()时，为了防止执行时间过程，批量操作时不再采用innodb_purge_batch_size(300)指定的值，而是采用20。


InnoDB的数据组织方式采用聚簇索引，也就是索引组织表，而二级索引采用(索引键值,主键键值)组合来唯一确定一条记录。
无论是聚簇索引，还是二级索引，每条记录都包含了一个DELETED-BIT位，用于标识该记录是否是删除记录；除此之外，聚簇索引还有两个系统列：DATA_TRX_ID，DATA_ROLL_PTR，分别表示产生当前记录项的事务ID以及指向当前记录的undo信息。



从聚簇索引行结构，与二级索引行结构可以看出，聚簇索引中包含版本信息(事务号+回滚指针)，二级索引不包含版本信息，二级索引项的可见性如何判断？？？？


InnoDB存储引擎在开始一个RR读之前，会创建一个Read View。Read View用于判断一条记录的可见性。Read View定义在read0read.h文件中，其中最主要的与可见性相关的属性如下：

class ReadView {
private:
  trx_id_t        m_low_limit_id;  //
};


mtr_commit(struct mtr_t*)                 提交一个mini-transaction，调用mtr_t::commit()
 |-mtr_t::Command::execute()              写redo-log，将脏页添加到flush-list，并释放占用资源
   |-mtr_t::Command::prepare_write()      准备写入日志
   | |-fil_names_write_if_was_clean()
   |-mtr_t::Command::finish_write()












妈的文件整理文件

http://www.sysdb.cn/index.php/2016/01/14/innodb-recovery/

http://www.cnblogs.com/liuhao/p/3714012.html


持续集成 https://www.zhihu.com/question/23444990




buf_flush_batch

## 简介

总体来说 double write 是为了在宕机或者掉电时提高可靠性，而牺牲了一点点写性能。介绍 double write 之前，有必要先了解一下 partial page write 问题。

InnoDB 中的默认页大小是 16KB，通过 innodb_page_size 变量定义，很多的操作，如数据校验、写入磁盘等，也是以页为单位进行。

而计算机硬件和操作系统的原子操作通常小于该值，一般为 512 字节，也就意味着，在极端情况下（如宕机、断电、OS Crash 等），往往并不能保证写入页的原子性。例如，16K 的数据，在写入 4K 时机器宕机，此时只有一部分写是成功的，这种情况下就是 partial page write 问题。

<!--
mysql在通过redolog恢复时，需要检查page的checksum，checksum就是pgae的最后事务号，发生partial page write问题时，page已经损坏，找不到该page中的事务号，就无法恢复。
-->

为了解决上述问题，采用两次写，此时需要额外添加两个部分，A) 内存中的两次写缓冲 (double write buffer)，大小为 2MB；B) 磁盘上共享表空间中连续的 128 页，大小也为 2MB。

### 工作过程

工作过程大致如下：

1. 当需要将缓冲池的脏页刷新到 data file 时，并不直接写到数据文件中，而是先拷贝至内存中的 double write buffer。
2. 接着从 double write buffer 分两次写入磁盘共享表空间中，每次写入 1MB，并马上调用 fsync 函数，同步到磁盘，避免缓冲带来的问题。
3. 第 2 步完成后，再将两次写缓冲区写入数据文件。

如下是执行示意图。

![how innodb double write works]({{ site.url }}/images/databases/mysql/innodb-double-write-works.jpg "how innodb double write works"){: .pull-center }

在这个过程中，第二步的 double write 是顺序写，所以开销并不大；而第三步，在将 double write buffer 写入各表空间文件，是离散写入；而 double write 实际引入的是第二步的开销。

### 恢复过程

有 double write 后，恢复时就简单多了，首先检查数据页，如果损坏，则尝试从 double write 中恢复数据；然后，检查 double writer 的数据的完整性，如果不完整直接丢弃，重新执行 redo log；如果 double write 的数据是完整的，用 double buffer 的数据更新该数据页，跳过该 redo log。

![how innodb double write recovery]({{ site.url }}/images/databases/mysql/innodb-double-write-recovery.jpg "how innodb double write recovery"){: .pull-center }

有些时候，并不一定需要 double write，例如，从机；有些文件系统 (如ZFS) 或者硬件也提供了类似的原子写入功能，因此可以关闭 double write 功能。

也即将 innodb_doublewrite 变量设置为 OFF，此时的写入过程大致如下。

![innodb double write closed]({{ site.url }}/images/databases/mysql/innodb-double-write-closed.png "innodb double write closed"){: .pull-center width="90%" }

### 配置参数

在 InnoDB 中，可以通过如下方式查看 double write 的状态。

{% highlight text %}
------ 查看是否启用了double write，以及相关参数
mysql> SHOW VARIABLES LIKE 'innodb_doublewrite%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| innodb_doublewrite            | ON    |
| innodb_doublewrite_batch_size | 120   |
+-------------------------------+-------+
2 rows in set (0.02 sec)

----- 可以查询double write的使用情况
mysql> SHOW STATUS LIKE 'innodb_dblwr_%';
mysql> SHOW STATUS LIKE 'innodb_dblwr_%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Innodb_dblwr_pages_written | 14615 |   从BP写入到dblwr的page数
| Innodb_dblwr_writes        | 636   |   写文件的次数
+----------------------------+-------+
2 rows in set (0.02 sec)
{% endhighlight %}

如上可以得到平均每次写操作合并页数为 ```Innodb_dblwr_pages_written/Innodb_dblwr_writes``` 。

## 源码解析

如上所述，一个 dblwr 有 2MB 也就是 128 pages，其中默认有 120 pages 用于批量刷新，可以直接通过 ```innodb_doublewrite_batch_size``` 变量设置，其包括了 BUF_FLUSH_LRU、BUF_FLUSH_LIST，剩下的 8 个页用于单个 page 的 flush。

{% highlight cpp %}
UNIV_INTERN buf_dblwr_t* buf_dblwr = NULL;          // 定义Double Write Buffer全局变量

struct buf_dblwr_t {
    ib_mutex_t  mutex;                 // 互斥量，用于保护first_free、write_buf
    ulint       block1;                // 第一个doubewrite块(64 pages)的page no
    ulint       block2;                // 第二个doublewrite块的page no
    ulint       first_free;            // 在write_buf中第一个空闲的位置，单位为UNIV_PAGE_SIZE
    ulint       b_reserved;            // 为batch flush预留的slot数
    os_event_t  b_event;               // 等待batch flush完成的事件
    ulint       s_reserved;            // 为单个page刷新预留的slot数
    os_event_t  s_event;               // 等待single flush完成的事件
    bool*       in_use;                // 标记一个slot是否被使用，只用于single page flush
    bool        batch_running;         // 当设置为TRUE时，表明有batch flush正在执行
    byte*       write_buf;             // dblwr在内存的缓存，以UNIV_PAGE_SIZE为单位对齐
    byte*       write_buf_unaligned;   // 未对齐的write_buf
    buf_page_t**    buf_block_arr;     // 存储已经cache到write_buf的block的指针
};
{% endhighlight %}

对于 FLUSH 操作，有三种类型。

* BUF_FLUSH_LRU<br>从 buffer pool 的 LRU 上扫描并刷新。
* BUF_FLUSH_LIST<br>从 buffer pool 的 FLUSH LIST 上扫描并刷新。
* BUF_FLUSH_SINGLE_PAGE<br>从 LRU 上只刷新一个 page，会通过 buf_dblwr_write_single_page() 来写一个 page 。

前两种属于 BATCH FLUSH，最后一种属于 SINGLE FLUSH，在 buf_flush_write_block_low() 函数中执行如下逻辑。

<!--
BUF_FLUSH_SINGLE_PAGE在几种情况下使用到：
1.buf_flush_or_remove_page
2.buf_flush_single_page_from_LRU，这在FREE LIST不够用时，IO-bound场景下，可能频繁调用到（buf_LRU_get_free_block）
3.buf_flush_page_try
-->


{% highlight text %}
buf_flush_page()                      将可以刷新的页写入到磁盘中
 |-buf_flush_write_block_low()
   |-buf_dblwr_write_single_page()    刷新类型为BUF_FLUSH_SINGLE_PAGE时
   |                                  会写入dblwr+sync，然后写入datafile+sync
   |
   |-buf_dblwr_add_to_batch()         批量写入
   |
   |-fil_flush()                      如果是同步，则刷新并等待执行完成
   |-buf_page_io_complete()



// 用于将一个page加入到double write buffer中，并完成写操作
void buf_dblwr_write_single_page(...) {
    ... ...
    n_slots = size - srv_doublewrite_batch_size; // 为单个page刷新预留的dblwr页数
    ... ...
    if (buf_dblwr->s_reserved == n_slots) {      // 当single slot已满时，直接重试
        ... ...
        goto retry;
    }

    for (i = srv_doublewrite_batch_size; i < size; ++i) { // 查找一个未使用的slot，并分配
        if (!buf_dblwr->in_use[i]) {
            break;
        }
    }
    ... ...
    fil_io(...)                 // 将单个page写入到double write buffer中

    fil_flush(TRX_SYS_SPACE);   // 将double write buffer数据写入到磁盘

    // 写入数据文件，注意该函数中会唤醒IO线程，等待写入完成，因此这是同步写操作
    buf_dblwr_write_block_to_datafile(bpage, sync);
}

// 将一个page加入到double write buffer中，如果batch满了，则刷double write buffer到磁盘
buf_dblwr_add_to_batch(bpage) {
    ... ...
    if (buf_dblwr->batch_running) { // 已经在做batch flush了，等待
        ... ...
        goto try_again;
    }

    if (buf_dblwr->first_free == srv_doublewrite_batch_size) { // batch已满
        mutex_exit(&(buf_dblwr->mutex));
        buf_dblwr_flush_buffered_writes();                     // 把dblwr的写到磁盘
        goto try_again;
    }
    ... ...
    buf_dblwr->buf_block_arr[buf_dblwr->first_free] = page;    // 将页拷贝到第buf_dblwr->first_free槽位

    ... ...    // 再次判断是否已满，满则调用buf_dblwr_flush_buffered_writes()
}

// 批量刷double write buffer的函数，只涉及batch flush的page
void buf_dblwr_flush_buffered_writes(void) {
    ... ..
    if (buf_dblwr->first_free == 0) { // 当前double write buffer为空，释放mutex，返回
        mutex_exit(&buf_dblwr->mutex);
        return;
    }

    if (buf_dblwr->batch_running) {  // 有线程在做batch flush，释放Mutex，重试
        ... ...
        os_event_wait_low(buf_dblwr->b_event, sig_count);
        goto try_again;
    }

    buf_dblwr->batch_running = true; // 设置为TRUE，避免并发写；并释放锁
    ... ...
    // 检查每一个将要写dblwr的block以及write_buf中的page是否被损坏或者LSN值是否正确
    buf_dblwr_check_block(block);
    buf_dblwr_check_page_lsn(write_buf + len2);

    // 将write_buf中的page分两次写入到文件中，先写block1，后写block2
    fil_io(OS_FILE_WRITE, true, TRX_SYS_SPACE, 0,
           buf_dblwr->block1, 0, len,
           (void*) write_buf, NULL);
    ... ...
    fil_flush(TRX_SYS_SPACE);  // 将double write buffer刷到磁盘


    for (ulint i = 0; i < first_free; i++) { // 逐个开始写数据文件
        buf_dblwr_write_block_to_datafile(
            buf_dblwr->buf_block_arr[i], false);
    }

    os_aio_simulated_wake_handler_threads(); // 唤醒IO线程
}
{% endhighlight %}

这里根据 flush 的类型来判断对应的函数，包括 buf_dblwr_write_single_page()、buf_dblwr_add_to_batch() 。

<!--
注意这里，在函数结束时并没有将batch_running设置为FALSE，因为这里对数据文件做的是异步写，设置标记位的工作留给了IO线程来完成
io_handler_thread-> fil_aio_wait-> buf_page_io_complete->buf_flush_write_complete->buf_dblwr_update()：
-->










FAQ系列 | 如何避免ibdata1文件大小暴涨

0、导读

    遇到InnoDB的共享表空间文件ibdata1文件大小暴增时，应该如何处理？

1、问题背景

用MySQL/InnoDB的童鞋可能也会有过烦恼，不知道为什么原因，ibdata1文件莫名其妙的增大，不知道该如何让它缩回去，就跟30岁之后男人的肚腩一样，汗啊，可喜可贺的是我的肚腩还没长出来，hoho~

正式开始之前，我们要先知道ibdata1文件是干什么用的。

ibdata1文件是InnoDB存储引擎的共享表空间文件，该文件中主要存储着下面这些数据：

        data dictionary
        double write buffer
        insert buffer/change buffer
        rollback segments
        undo space
        Foreign key constraint system tables

另外，当选项 innodb_file_per_table = 0 时，在ibdata1文件中还需要存储 InnoDB 表数据&索引。ibdata1文件从5.6.7版本开始，默认大小是12MB，而在这之前默认大小是10MB，其相关选项是 innodb_data_file_path，比如我一般是这么设置的：

    innodb_data_file_path = ibdata1:1G:autoextend

当然了，无论是否启用了 innodb_file_per_table = 1，ibdata1文件都必须存在，因为它必须存储上述 InnoDB 引擎所依赖&必须的数据，尤其是上面加粗标识的 rollback segments 和 undo space，它俩是引起 ibdata1 文件大小增加的最大原因，我们下面会详细说。
2、原因分析

我们知道，InnoDB是支持MVCC的，它和ORACLE类似，采用 undo log、redo log来实现MVCC特性的。在事务中对一行数据进行修改时，InnoDB 会把这行数据的旧版本数据存储一份在undo log中，如果这时候有另一个事务又要修改这行数据，就又会把该事物最新可见的数据版本存储一份在undo log中，以此类推，如果该数据当前有N个事务要对其进行修改，就需要存储N份历史版本（和ORACLE略有不同的是，InnoDB的undo log不完全是物理block，主要是逻辑日志，这个可以查看 InnoDB 源码或其他相关资料）。这些 undo log 需要等待该事务结束后，并再次根据事务隔离级别所决定的对其他事务而言的可见性进行判断，确认是否可以将这些 undo log 删除掉，这个工作称为 purge（purge 工作不仅仅是删除过期不用的 undo log，还有其他，以后有机会再说）。

那么问题来了，如果当前有个事务中需要读取到大量数据的历史版本，而该事务因为某些原因无法今早提交或回滚，而该事务发起之后又有大量事务需要对这些数据进行修改，这些新事务产生的 undo log 就一直无法被删除掉，形成了堆积，这就是导致 ibdata1 文件大小增大最主要的原因之一。这种情况最经典的场景就是大量数据备份，因此我们建议把备份工作放在专用的 slave server 上，不要放在 master server 上。

另一种情况是，InnoDB的 purge 工作因为本次 file i/o 性能是在太差或其他的原因，一直无法及时把可以删除的 undo log 进行purge 从而形成堆积，这是导致 ibdata1 文件大小增大另一个最主要的原因。这种场景发生在服务器硬件配置比较弱，没有及时跟上业务发展而升级的情况。

比较少见的一种是在早期运行在32位系统的MySQL版本中存在bug，当发现待 purge 的 undo log 总量超过某个值时，purge 线程直接放弃抵抗，再也不进行 purge 了，这个问题在我们早期使用32位MySQL 5.0版本时遇到的比较多，我们曾经遇到这个文件涨到100多G的情况。后来我们费了很大功夫把这些实例都迁移到64位系统下，终于解决了这个问题。

最后一个是，选项 innodb_data_file_path 值一开始就没调整或者设置很小，这就必不可免导致 ibdata1 文件增大了。Percona官方提供的 my.cnf 参考文件中也一直没把这个值加大，让我百思不得其解，难道是为了像那个经常被我吐槽的xx那样，故意留个暗门，好方便后续帮客户进行优化吗？（我心理太阴暗了，不好不好~~）

稍微总结下，导致ibdata1文件大小暴涨的原因有下面几个：

        有大量并发事务，产生大量的undo log；
        有旧事务长时间未提交，产生大量旧undo log；
        file i/o性能差，purge进度慢；
        初始化设置太小不够用；
        32-bit系统下有bug。

稍微题外话补充下，另一个热门数据库 PostgreSQL 的做法是把各个历史版本的数据 和 原数据表空间 存储在一起，所以不存在本案例的问题，也因此 PostgreSQL 的事务回滚会非常快，并且还需要定期做 vaccum 工作（具体可参见PostgreSQL的MVCC实现机制，我可能说的不是完全正确哈）
3、解决方法建议

看到上面的这些问题原因描述，有些同学可能觉得这个好办啊，对 ibdata1 文件大小进行收缩，回收表空间不就结了吗。悲剧的是，截止目前，InnoDB 还没有办法对 ibdata1 文件表空间进行回收/收缩，一旦 ibdata1 文件的肚子被搞大了，只能把数据先备份后恢复再次重新初始化实例才能恢复原先的大小，或者把依次把各个独立表空间文件备份恢复到一个新实例中，除此外，没什么更好的办法了。

当然了，这个问题也并不是不能防范，根据上面提到的原因，相应的建议对策是：

        升级到5.6及以上（64-bit），采用独立undo表空间，5.6版本开始就支持独立的undo表空间了，再也不用担心会把 ibdata1 文件搞大；
        初始化设置时，把 ibdata1 文件至少设置为1GB以上；
        增加purge线程数，比如设置 innodb_purge_threads = 8；
        提高file i/o能力，该上SSD的赶紧上；
        事务及时提交，不要积压；
        默认打开autocommit = 1，避免忘了某个事务长时间未提交；
        检查开发框架，确认是否设置了 autocommit=0，记得在事务结束后都有显式提交或回滚。



关于MySQL的方方面面大家想了解什么，可以直接留言回复，我会从中选择一些热门话题进行分享。 同时希望大家多多转发，多一些阅读量是老叶继续努力分享的绝佳助力，谢谢大家 :)

最后打个广告，运维圈人士专属铁观音茶叶微店上线了，访问：http://yejinrong.com 获得专属优惠






## 参考

XtraDB: The Top 10 enhancements
https://www.percona.com/blog/2009/08/13/xtradb-the-top-10-enhancements/

https://forums.cpanel.net/threads/innodb-corruption-repair-guide.418722/

http://www.itpub.net/thread-2083877-1-1.html
{% highlight text %}
{% endhighlight %}

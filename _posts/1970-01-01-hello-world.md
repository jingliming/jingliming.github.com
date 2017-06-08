---
title: Hello World !!!
layout: post
comments: true
language: chinese
usemath: true
category: [misc]
keywords: hello world,示例,sample,markdown
description: 简单记录一下一些与 Markdown 相关的内容，包括了一些使用模版。
---

Hi, the world, I'm coooooooooming.

Oooops, just examples, ignore me, darling.

<!-- more -->

![hello world logo]({{ site.url }}/images/misc/hello-world-logo.jpg "hello world logo"){: .pull-center }

## 配色

<style type="text/css">
<!--
#colorbox span{display:block;float:left;height:195px;}
-->
</style>

静思

<div id="colorbox">
    <span style="width:125.2px;background-color: #81C2D6" alt="81C2D6">HEX<br>#81C2D6<br>RGB<br>129.194.214<br>CMYK<br>10.9.0.16</span>
    <span style="width:125.2px;background-color: #8192D6" alt="8192D6">HEX<br>#8192D6<br>RGB<br>129.146.214<br>CMYK<br>40.32.0.16</span>
    <span style="width:125.2px;background-color: #D9B3E6" alt="D9B3E6">HEX<br>#D9B3E6<br>RGB<br>217.179.230<br>CMYK<br>6.22.0.10</span>
    <span style="width:125.2px;background-color: #DCF7A1" alt="DCF7A1">HEX<br>#DCF7A1<br>RGB<br>220.247.161<br>CMYK<br>11.0.35.3</span>
    <span style="width:125.2px;background-color: #83FCD8" alt="83FCD8">HEX<br>#83FCD8<br>RGB<br>131.152.216<br>CMYK<br>48.0.14.1</span>
</div>
<div class="clearfix"></div>

## Heading 2，目录 2

### Heading 3， 目录 3

#### Heading 4，目录 4

##### Heading 5，目录 5

###### Heading 6， 目录 6


## MISC

### Separator，分割线

下面的效果是相同的。

* * *

***

*****

- - -

---------------------------------------




### The Fonts， 字体设置

*This is emphasized 斜体*.   _This is also emphasized 还是斜体_.

**Strong font 粗体** __Also strong font 还是粗体__

Water is H<sub>2</sub>O. 4<sup>2</sup>=16. 上标、下标测试。

Code Use the `printf()` function，代码模块。

Code Use the <code>printf()</code> function，与上面功能一样。

``There is a literal backtick (`) here.``，额，还是代码模块。

The New York Times <cite>(That’s a citation)</cite>. 引用测试，和斜体相似。

This is <u>Underline</u>. 下划线。


### Code Snippets，代码高亮显示

Syntax highlighting via Pygments. css java sh c gas asm cpp c++

{% highlight css linenos %}
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
{% endhighlight %}


{% highlight c %}
int main ()
{
   return 0;
}
{% endhighlight %}

Non Pygments code example

    <div id="awesome">
        <p>This is great isn't it?</p>
    </div>

















### Block Quote， 段引用

下面时关于段引用的测试。

单段的引用。Just one paragraph.

> My conclusion after the trip was "well, now I know that there's at least one company in the world that can succeed with the waterfall model" and I decided to stop bashing the waterfall model as hard as I usually do. Now, however, with all the problems Toyota are having, I'm starting to reconsider.q q q q q q q q q q q q q q q<from>kkkkk</from>

多段的引用，one more paragraphs.


> My conclusion after the trip was "well, now I know that there's at least one company in the world that can succeed with the waterfall model" and I decided to stop bashing the waterfall model as hard as I usually do. Now, however, with all the problems Toyota are having, I'm starting to reconsider.
>
> My conclusion after the trip was "well, now I know that there's at least one company in the world that can succeed with the waterfall model" and I decided to stop bashing the waterfall model as hard as I usually do. Now, however, with all the problems Toyota are having, I'm starting to .q q q q q

单段，但较为复杂的引用。

> My conclusion after the trip was "well,
> now I know that there's at least one company in the world
> that can succeed with the waterfall model" and I decided to
> stop bashing the waterfall model as hard as I usually do.
> Now, however, with all the problems Toyota are having, I'm starting to reconsider.

嵌套引用。

> My conclusion after the trip was "well,
> now I know that there's at least one company in the world
> > that can succeed with the waterfall model" and I decided to
> stop bashing the waterfall model as hard as I usually do.
> Now, how ever, with all the problems Toyota are having, I'm starting to re consider.







### Unordered Lists，无序列表

如下是三种不同的表达方式。

#### Unordered Lists 1

* Item one
* Item two
* Item three

#### Unordered Lists 2

+ Item one
+ Item two
+ Item three

#### Unordered Lists 3

- Item one
- Item two
- Item three

#### 其它

如下的各个列表项中，各个项之间表示为段落，而之前的不是，也就是说添加了 ```<p></p>``` ，所以现在看起来各个段之间空隙有点大。

- Item one

- Item two

- Item three

### Ordered Lists，有序列表

有序表的表达方式，只与顺序相关，而与列表前的数字无关。

#### Ordered Lists 1

1. Item one
   1. sub item one
   2. sub item two
   3. sub item three
2. Item two

#### Ordered Lists 2

1. Item one
1. Item two
1. Item three

#### Ordered Lists 3

3. Item one
9. Item two
5. Item three

### Lists Tips，列表补记

列表项目标记通常是放在最左边，但是其实也可以缩进，最多 3 个空格，项目标记后面则一定要接着至少一个空格或制表符。

*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.

如下显示相同。

*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
Suspendisse id sem consectetuer libero luctus adipiscing.

如下是在同一列表中，显示两个段落。

1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.

    This is the second paragraph in the list item. You're
only required to indent the first line. Lorem ipsum dolor
sit amet, consectetuer adipiscing elit.

### Tables, 表格

kramdown 默认支持表格，只需要设置好 ```table thead tbody th tr td``` 对应的属性即可。

|head1 head1 head1|head2 head2 head2|head3 head3 head3|head4 head4 head4|
|---|:---|:---:|---:|
|row1text1|row1text3|row1text3|row1text4|
|row2text1|row2text3|row2text3|row2text4|

<!--
dog | bird | cat
----|------|----
foo | foo  | foo
bar | bar  | bar
baz | baz  | baz
-->

### MathJax, 数学表达式

如下是一个数学表达式。

$$
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
$$

段内插入LaTeX代码是这样的：$\exp(-\frac{x^2}{2})$，试试看看吧

### Pictures，图片显示

![If the picture doesnt exist]({{ site.url }}/images/linux-liberty.png "an example picture"){: .pull-left}

<div class="clearfix"></div>

![aaaaa]{: .pull-right width="20%"}

<div class="clearfix"></div>

[aaaaa]:    /images/linux-liberty.png    "MSN Search"


### Reference，引用

如下是一个简单的链接 [an example](http://example.com/ "Title")，当然也可以使用网站的相对路径 [About Me](/about/) 。

也可以将网站的引用与 URL 分别区分开，如下是其中的示例，而且不区分大小写，[RTEMS] [1]、[Linux] [2]、[GUN][GUN]、[GUN][gun] 。

<!-- the following can occur on anywhere -->
[1]: http://www.rtems.com                              "A Real Time kernel"
[2]: http://www.Linux.com                              'Linux'
[Gun]: http://www.gun.com                              (GUN)


<!--

<figure class="highlight"><pre><code class="language-css" data-lang="css"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4
5
6
7
8
9
10</pre></td><td class="code"><pre><span class="nf">#container</span> <span class="p">{</span>
  <span class="nl">float</span><span class="p">:</span> <span class="nb">left</span><span class="p">;</span>
  <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span> <span class="m">-240px</span> <span class="m">0</span> <span class="m">0</span><span class="p">;</span>
  <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
<span class="p">}</span>
<span class="nf">#container</span> <span class="p">{</span>
  <span class="nl">float</span><span class="p">:</span> <span class="nb">left</span><span class="p">;</span>
  <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span> <span class="m">-240px</span> <span class="m">0</span> <span class="m">0</span><span class="p">;</span>
  <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
<span class="p">}</span><span class="w">
</span></pre></td></tr></tbody></table></code></pre></figure>


①
②
③
④
⑤
⑥
⑦
⑧
⑨
⑩
⑪
⑫
⑬
⑭
⑮
⑯
⑰
⑱
⑲
⑳

❶
❷
❸
❹
❺
❻
❼
❽
❾
❿
⓿
➊
➋
➌
➍
➎
➏
➐
➑
➒
➓


用pt-table-checksum校验数据一致性
http://www.nettedfish.com/blog/2013/06/04/check-replication-consistency-by-pt-table-checksum/

Choosing MySQL High Availability Solutions

Database Daily Ops Series: GTID Replication
https://www.percona.com/blog/2016/11/10/database-daily-ops-series-gtid-replication/

http://hatemysql.com/?p=526
http://mysqllover.com/?p=594
http://in355hz.iteye.com/blog/1770401

binlog文件格式
http://www.jianshu.com/p/c16686b35807
http://slideplayer.com/slide/6146721/
http://miguelangelnieto.net/index.php/how-to-createrestore-a-slave-using-gtid-replication-in-mysql-5-6/

???支持基于所有数据库、指定数据库、指定表的复制。

默认使用基于GTID的异步复制，同时支持半同步复制 (semi-synchronous)，基于 NDB 的同步复制以及延迟复制 (Delayed Replication)




### NC
socat加强版
http://brieflyx.me/2015/linux-tools/socat-introduction/


### GCC
http://www.jianshu.com/p/dd425b9dc9db
http://filwmm1314.blog.163.com/blog/static/218259192012121225132/
http://www.cnblogs.com/respawn/archive/2012/07/09/2582078.html

### TSD

在多线程程序中，所有线程共享程序中的变量，如果每个线程需要保存独自的数据，例如每个线程维护一个链表，但是通过相同的函数处理，这就是 Thread Specific Data 的作用。如下介绍 TSD 的使用方法：

1. 声明一个 pthread_key_t 类型的全局变量；
2. 通过 pthread_key_create() 函数创建 TSD，实际就是分配一个实例，并将其赋值给 pthread_key_t 变量，所有的线程都可以通过该变量访问，这就相当于提供了同名而不同值的全局变量；
3. 调用  pthread_setspcific()、pthread_getspecific() 存储或者获取各个线程特有的值；

TSD的实现详见： https://www.ibm.com/developerworks/cn/linux/thread/posix_threadapi/part2/

int pthread_key_create(pthread_key_t *key, void (*destructor)(void*));
int pthread_setspecific(pthread_key_t key, const void *value);
void *pthread_getspecific(pthread_key_t key);



### innodb_locks_unsafe_for_binlog 参数相关
另外设置 innodb_locks_unsafe_for_binlog=1 ,binlog也要设为row格式。
https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog
innodb默认使用了next-gap算法，这种算法结合了index-row锁和gap锁。正因为这样的锁算法，innodb在可重复读这样的默认隔离级别上，可以避免幻象的产生。innodb_locks_unsafe_for_binlog最主要的作用就是控制innodb是否对gap加锁。
注意该参数如果是enable的，则是unsafe的，此时gap不会加锁；反之，如果disable掉该参数，则gap会加锁。当然对于一些和数据完整性相关的定义，如外键和唯一索引（含主键）需要对gap进行加锁，那么innodb_locks_unsafe_for_binlog的设置并不会影响gap是否加锁。
在5.1.15的时候，innodb引入了一个概念叫做“semi-consistent”，这样会在innodb_locks_unsafe_for_binlog的状态为ennable时在一定程度上提高update并发性。
https://yq.aliyun.com/articles/62372



### FLUSH TABLES WITH READ LOCK
FLUSH TABLES WITH READ LOCK命令的作用是关闭所有打开的表，会将所有脏页刷新到磁盘，同时对所有数据库中的表都加一个全局读锁，直到显示执行UNLOCK TABLES，才释放持有的读锁。

该操作常用于获取一致性数据备份。

http://aklaus.blog.51cto.com/9724632/1656991
http://blog.csdn.net/zbszhangbosen/article/details/7434181
http://www.cnblogs.com/cchust/p/4603599.html
http://www.cnblogs.com/sunss/archive/2012/02/02/2335960.html
http://www.cnblogs.com/zhenghongxin/p/5527527.html
http://myrock.github.io/2014/11/20/mysql-waiting-for-table-flush/
http://blog.csdn.net/arkblue/article/details/27376991
https://www.percona.com/blog/2012/03/23/how-flush-tables-with-read-lock-works-with-innodb-tables/
https://www.percona.com/blog/2010/04/24/how-fast-is-flush-tables-with-read-lock/

加锁过程是什么样的，为什么会阻塞现有的SELECT查询？

http://www.penglixun.com/tech/database/the_process_of_mysqldump.html
http://www.cnblogs.com/digdeep/p/4947694.html
http://www.imysql.com/2008_10_24_deep_into_mysqldump_options
http://www.cnblogs.com/zhoujinyi/p/5789465.html
https://yq.aliyun.com/articles/59326


#### sql_slave_skip_counter=N
http://dinglin.iteye.com/blog/1236330
http://xstarcd.github.io/wiki/MySQL/MySQL_replicate_error.html

#### FOR UPDATE/
SELECT ... FOR UPDATE 用于对查询的数据添加IX锁(意向排它锁)，此时，其它会话也就无法在这些记录上添加任何的S锁或X锁，通常使用的场景是为了防止在低隔离级别下出现幻读现象，用于保证 “本事务看到的是数据库存储的最新数据，并且看到的数据只能由本事务修改”。

InnoDB默认使用快照读，使用 FOR UPDATE 之后不会阻塞其它会话的快照读，当然会阻塞lock in share mode和for update这种显示加锁的查询操作。

CREATE TABLE `foobar` (
  `id` int(11) NOT NULL,
  `col` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO foobar VALUES(1, 11),(2, 22);
set session transaction isolation level REPEATABLE READ;
set autocommit=0;
select * from foobar where col = 11; **FOR UPDATE**

set session transaction isolation level REPEATABLE READ;
set autocommit=0;
select * from foobar where col = 11;
update foobar set col = 33 where col=11;

select * from foobar where col = 11;
update foobar set col = col*2 where col=11;


SELECT ... LOCK IN SHARE MODE对于查询的数据添加IS(意向共享锁)，此时，其它会话可以读取这些记录，也可以继续添加IS锁，但是无法修改这些记录直到该事务执行完成。

通常用于两张存在关联关系表的写操作，拿mysql官方文档的例子来说，一个表是child表，一个是parent表，假设child表的某一列child_id映射到parent表的c_child_id列，那么从业务角度讲，此时我直接insert一条child_id=100记录到child表是存在风险的，因为刚insert的时候可能在parent表里删除了这条c_child_id=100的记录，那么业务数据就存在不一致的风险。正确的方法是再插入时执行select * from parent where c_child_id=100 lock in share mode,锁定了parent表的这条记录，然后执行insert into child(child_id) values (100)就ok了。

http://blog.csdn.net/cug_jiang126com/article/details/50544728
http://chenzhou123520.iteye.com/blog/1860954
https://www.percona.com/blog/2006/08/06/select-lock-in-share-mode-and-for-update/


#### 复制过滤可能产生的异常

http://keithlan.github.io/2015/11/02/mysql_replicate_rule/

#### BLOG和TEXT区别
BLOB和TEXT分别用于保存二进制和非二进制字符串，保存数据可变，都可以分为四类：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB ，TEXT与之类似，只是其容纳长度有所不同；两者区别如下：

* BLOB为二进制字符串，TEXT为非二进制字符串；
* BLOG列没有字符集，并且排序和比较基于列值字节的数值；TEXT列有字符集，并且根据字符集的规则进行排序和比较；
* 两种类型的DML不存在大小写转换，在非严格模式下超出最大长度时会截断并产生告警；严格模式则会报错；

两者可以分别视为VARCHAR和VARBINARY，但是有如下区别：
* BLOB和TEXT列不能有默认值；

当保存或检索BLOB和TEXT列的值时不删除尾部空格。(这与VARBINARY和VARCHAR列相同）.

对于BLOB和TEXT列的索引，必须指定索引前缀的长度。对于CHAR和VARCHAR，前缀长度是可选的.

LONG和LONG VARCHAR对应MEDIUMTEXT数据类型。这是为了保证兼容性。如果TEXT列类型使用BINARY属性，将为列分配列字符集的二元校对规则.

MySQL连接程序/ODBC将BLOB值定义为LONGVARBINARY，将MySQL TEXT值定义为LONGVARCHAR。由于BLOB和TEXT值可能会非常长，使用它们时可能遇到一些约束.

BLOB或TEXT对象的最大大小由其类型确定，但在客户端和服务器之间实际可以传递的最大值由可用内存数量和通信缓存区大小确定。你可以通过更改max_allowed_packet变量的值更改消息缓存区的大小，但必须同时修改服务器和客户端程序。例如，可以使用 MySQL和MySQLdump来更改客户端的max_allowed_packet值.


create table foobar (
 
) engine=innodb;

mysql>CREATE TABLE foobar (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  book CHAR(10) DEFAULT NULL,
  msg BLOB,
  ) ENGINE=InnoDB;
http://blog.sina.com.cn/s/blog_4f925fc30102edg8.html
http://imysql.com/2014/09/28/mysql-optimization-case-blob-stored-in-innodb-optimization.shtml
http://www.qttc.net/201207121.html
http://53873039oycg.iteye.com/blog/2002448



#### 高可用
循环复制
http://xiezhenye.com/2015/08/%E8%AE%B0%E4%B8%80%E6%AC%A1-mysql-%E5%BE%AA%E7%8E%AF%E5%A4%8D%E5%88%B6.html
http://www.cnblogs.com/cchust/p/5450510.html
http://imysql.com/2015/09/14/solutions-of-mysql-ha.shtml
http://blog.csdn.net/yangbutao/article/details/8807813
http://www.jianshu.com/p/cc6746ac4fc2
http://raffaelexr.blog.51cto.com/8555551/1747784
http://wenson.iteye.com/blog/2263956

#### VIP
http://www.cnblogs.com/pangguoping/p/5721517.html


##### MySQL Cache

sql_cache意思是说，查询的时候使用缓存。

sql_no_cache意思是查询的时候不适用缓存。

sql_buffer_result意思是说，在查询语句中，将查询结果缓存到临时表中。

这三者正好配套使用。sql_buffer_result将尽快释放表锁，这样其他sql就能够尽快执行。

使用 FLUSH QUERY CACHE 命令，你可以整理查询缓存，以更好的利用它的内存。这个命令不会从缓存中移除任何查询。FLUSH TABLES 会转储清除查询缓存。
RESET QUERY CACHE 使命从查询缓存中移除所有的查询结果。

--------------------------------------------------

那么MySQL到底是怎么决定到底要不要把查询结果放到查询缓存中呢？

是根据query_cache_type这个变量来决定的。

这个变量有三个取值：0,1,2，分别代表了off、on、demand。

意思是说，如果是0，那么query cache 是关闭的。如果是1，那么查询总是先到查询缓存中查找，除非使用了sql_no_cache。如果是2，那么，只有使用 sql_cache的查询，才会去查询缓存中查找。


###
ysql [(none)]>grant super on db1.* to 'dtstack'@'%';
ERROR 1221 (HY000): Incorrect usage of DB GRANT and GLOBAL PRIVILEGES        --因为super权限是管理级别的权限(super ,process,file)，不能够指定某个数据库on 后面必须跟*.*

正确用法：
mysql [(none)]>grant super on *.* to 'dtstack'@'%';
Query OK, 0 rows affected (0.01 sec)
释：
1.super权限可以使用change master to 语句
2.kill ， mysqladmin kill  kill_threads
3.purge binary logs
    EG：
        wjj@(www.dtstack.com) [(none)]>show binary logs;
          mysql [(none)]>purge binary logs to 'mysql-bin.000005';  --解释：删除MySQLbinlog日志，删除mysql-bin.000005之前的，不包括他本身
          Query OK, 0 rows affected (0.03 sec
          mysql [(none)]>show binary logs;
       
4.可以设置全局参数模式set global xx
5.mysqladmin debug  启动或禁用日志记录
6.有权限设置relad_only
7.连接数达到max_connections上限时无法创建连接时，拥有super权限的用户可以创建一个连接

网络设置，包括了如何设置服务端+客户端、广播、多播等。
https://collectd.org/wiki/index.php/Networking_introduction
写入RRD文件，包括了Collectd对RRD的优化，以及其中一篇RRD优化介绍的文章
http://oss.oetiker.ch/rrdtool-trac/wiki/TuningRRD
https://collectd.org/wiki/index.php/Inside_the_RRDtool_plugin
惊群问题讨论
http://www.voidcn.com/blog/liujiyong7/article/p-377809.html
linux时间相关结构体和函数整理
http://www.cnblogs.com/zhiranok/archive/2012/01/15/linux_c_time_struct.html
Heap数据结构(栈)
http://www.cnblogs.com/gaochundong/p/binary_heap.html
http://www.cnblogs.com/skywang12345/p/3610187.html


AVL数
https://courses.cs.washington.edu/courses/cse373/06sp/handouts/lecture12.pdf
https://www.cise.ufl.edu/~nemo/cop3530/AVL-Tree-Rotations.pdf
http://www.cnblogs.com/zhoujinyi/p/6497231.html

https://dev.mysql.com/doc/refman/5.7/en/backup-policy.html
https://dev.mysql.com/doc/refman/5.7/en/point-in-time-recovery.html

https://www.unixhot.com/page/ops




对于GROUP BY、ORDER BY、DISTINCT类似的SQL，可能会用到临时表，通常在内存中使用MEMORY存储引擎，或者磁盘上使用MyISAM存储引擎。一般，当超过了tmp_table_size大小之后，会在磁盘上创建，另外，对于
https://www.percona.com/blog/2007/08/16/how-much-overhead-is-caused-by-on-disk-temporary-tables/
https://www.percona.com/blog/2007/08/18/how-fast-can-you-sort-data-with-mysql/





http://halobates.de/memorywaste.pdf

BuildBot


shell的通配符介绍
http://www.cnblogs.com/chengmo/archive/2010/10/17/1853344.html

### fnmatch()

就是判断字符串是不是符合 pattern 所指的结构，这里的 pattern 是 shell wildcard pattern，其中部分匹配行为可以通过 flags 配置，详见 man 3 fnmatch。

int fnmatch(const char *pattern, const char *string, int flags);


#include <stdio.h>
#include <stdlib.h>
#include <fnmatch.h>
#include <sys/types.h>
#include <dirent.h>

int main(int argc, char *argv[])
{
    int ret;
    char *pattern = "*.c";
    DIR *dir;
    struct dirent *entry;

    if ((dir = opendir("/tmp")) == NULL) {
        perror("opendir()");
        exit(EXIT_FAILURE);
    }
    while ((entry = readdir(dir)) != NULL) { // 逐个获取文件夹中文件
        ret = fnmatch(pattern, entry->d_name, FNM_PATHNAME|FNM_PERIOD);
        if (ret == 0) {         //符合pattern的结构
            printf("%s\n", entry->d_name);
        } else if (ret == FNM_NOMATCH){
            continue ;
        } else {
            printf("error file=%s\n", entry->d_name);
        }
    }
    closedir(dir);
    return 0;
}

wordexp()

按照 Shell-Style Word Expansion 扩展将输入字符串拆分，返回的格式为 wordexp_t 变量，其中包括了三个变量，两个比较重要的是：A) we_wordc 成员数；B) we_wodv 数组。

注意，在解析时会动态分配内存，所以在执行完 wordexp() 后，需要执行 wordfree()；另外，如果遇到内存不足会返回 WRDE_NOSPACE 错误，此时可能已经分配了部分地址，所以仍需要执行 wordfree() 。

1) 按照空格解析；2) 正则表达式；3) 环境变量。

#include <stdio.h>
#include <stdlib.h>
#include <wordexp.h>

int main(int argc, char **argv)
{
    int i, ret;
    wordexp_t p;

    ret = wordexp("foo bar $SHELL *[0-9].c *.c", &p, 0);
    if (ret == WRDE_NOSPACE) {
        wordfree(&p);
        exit(EXIT_FAILURE);
    } else if (ret != 0) {
        exit(EXIT_FAILURE);
    }
    for (i = 0; i < p.we_wordc; i++)
        printf("%s\n", p.we_wordv[i]);
    wordfree(&p);
    exit(EXIT_SUCCESS);
}

http://www.gnu.org/software/libc/manual/html_node/Word-Expansion.html


### qsort()

只用于数组的排序，对于链表等无效。

void qsort(void *base, size_t nitems, size_t size, int (*compar)(const void *, const void*));

base  : 数组的基地址
nitems: 数组包含的元素；
size  : 每个元素的大小；
compar: 比较函数；

#include <stdio.h>
#include <stdlib.h>

int cmpfunc (const void *a, const void *b)
{
   return ( *(int*)a - *(int*)b );
}

int main()
{
   int n;
   int values[] = { 88, 56, 100, 2, 25 };

   printf("Before sorting the list is: \n");
   for( n = 0 ; n < 5; n++ ) {
      printf("%d ", values[n]);
   }
   putchar('\n');
   qsort(values, 5, sizeof(int), cmpfunc);
   printf("After sorting the list is: \n");
   for( n = 0 ; n < 5; n++ ) {
      printf("%d ", values[n]);
   }
   putchar('\n');

   return(0);
}


非常经典的《Linux平台下的漏洞分析入门 》
https://github.com/1u4nx/Exploit-Exercises-Nebula
原文在这里
https://www.mattandreko.com/

http://hustcat.github.io/iostats/
http://ykrocku.github.io/blog/2014/04/11/diskstats/
http://www.udpwork.com/item/12931.html

FIXME:
  linux-monitor-io.html
/proc/diskstats 中包括了主设备号、次设备号和设备名称，剩余的各个字段的含义简单列举如下，详细可以查看内核文档 [I/O statistics fields](https://www.kernel.org/doc/Documentation/iostats.txt) 。

可以通过 grep diskstats 找到对应内核源码实现在 diskstats_show()@block/genhd.c 中。

获取源码 diskstats_show() + struct disk_stats 。

可以看到是通过 part_round_stats() 函数获取每个磁盘的最新统计信息，通过 struct hd_struct 中的 struct disk_stats *dkstats 结构体保存，然后利用 part_stat_read() 函数统计各个 CPU 的值 (如果是多核)。


在 Bash 编程时，经常需要切换目录，可以通过 pushd、popd、dirs 命令切换目录。

pushd  切换到参数指定的目录，并把原目录和当前目录压入到一个虚拟的堆栈中，不加参数则在最近两个目录间切换；
popd   弹出堆栈中最近的目录；
dirs   列出当前堆栈中保存的目录列表；
  -v 在目录前添加编号，每行显示一个目录；
  -c 清空栈；

切换目录时，会将上次目录保存在 $OLDPWD 变量中，与 "-" 相同，可以通过 cd - 切换回上次的目录。



set -o history 开启bash的历史功能

判断目录是否存在，如果目录名中有'-'则需要进行转义。

dir="/tmp/0\-test"
if [ ! -d "${dir}" ]; then
  mkdir /myfolder
fi


c_avl_tree_t *c_avl_create(int (*compare)(const void *, const void *));
入参：
  比较函数，类似strcmp()；
实现：
  1. 保证 compare 有效，非 NULL；
  2. 申请内存，部分结构体初始化。
返回：
  成功返回结构体指针；参数异常或者没有内存，返回 NULL；

int c_avl_insert(c_avl_tree_t *t, void *key, void *value);
返回：
  -1：内存不足；
  0： 节点写入正常；
  1:  节点已经存在；

int c_avl_get(c_avl_tree_t *t, const void *key, void **value);
调用者保证 t 存在 (非NULL)。
返回：
  -1：对象不存在；
  0： 查找成功，对应的值保存在value中；

int c_avl_remove(c_avl_tree_t *t, const void *key, void **rkey, void **rvalue);
返回：
  -1：对象不存在；


_remove()
search()
rebalance()
verify_tree()


安全渗透所需的工具
https://wizardforcel.gitbooks.io/daxueba-kali-linux-tutorial/content/2.html






1/0 = &infin;
log (0) = -&infin;
sqrt (-1) = NaN

infinity (无穷)、NaN (非数值)

除了 infin 自身和 NaN 之外，infin (正无穷) 大于任何值；而 NaN 不等于任何值，包括其自身，而且与 <, >, <=, >= 使用时将会报错。







如果有 group 采用相同的 group-id，也就是有 alias group ，那么在删除某个

groupdel: cannot remove the primary group of user 'monitor'

----- 将原用户修改为其它分组
# usermod -g sys monitor
----- 删除分组
# groupdel test
----- 修改回来
# usermod -g monitor monitor












http://fengyuzaitu.blog.51cto.com/5218690/1616268
http://www.runoob.com/python/os-statvfs.html
http://blog.csdn.net/papiping/article/details/6980573
http://blog.csdn.net/hepeng597/article/details/8925506






jinyang ALL=(root) NOPASSWD: ALL






shell版本号比较
http://blog.topspeedsnail.com/archives/3999
https://www.netfilter.org/documentation/HOWTO/NAT-HOWTO-6.html
man 3 yum.conf 确认下YUM配置文件中的变量信息
https://unix.stackexchange.com/questions/19701/yum-how-can-i-view-variables-like-releasever-basearch-yum0


FORTIFY.Key_Management--Hardcoded_Encryption_Key    strcasecmp("Interval", child->key)

int lt_dlinit (void);
  初始化，在使用前调用，可以多次调用，正常返回 0 ；
const char * lt_dlerror (void);
  返回最近一次可读的错误原因，如果没有错误返回 NULL；
void * lt_dlsym (lt_dlhandle handle, const char *name);
  返回指向 name 模块的指针，如果没有找到则返回 NULL 。
lt_dlhandle lt_dlopen (const char *filename);
  加载失败返回 NULL，多次加载会返回相同的值；
int lt_dlclose (lt_dlhandle handle);
  模块的应用次数减一，当减到 0 时会自动卸载；成功返回 0 。

https://github.com/carpedm20/awesome-hacking
http://jamyy.us.to/blog/2014/01/5800.html





DJBHash()
 - hash += (hash << 5) + (*str++);
 + hash = ((hash << 5) + hash)  + (*str++);
https://www.byvoid.com/zhs/blog/string-hash-compare
http://www.partow.net/programming/hashfunctions/index.html
UDP Socket编程
http://www.binarytides.com/programming-udp-sockets-c-linux/
Page Cache
https://www.thomas-krenn.com/en/wiki/Linux_Page_Cache_Basics
一个由进程内存布局异常引起的问题
http://www.cnblogs.com/catch/p/6370859.html
Page Cache, the Affair Between Memory and Files
http://duartes.org/gustavo/blog/post/page-cache-the-affair-between-memory-and-files/
https://www.quora.com/What-is-the-major-difference-between-the-buffer-cache-and-the-page-cache
从free到page cache
http://www.cnblogs.com/hustcat/archive/2011/10/27/2226995.html
FIXME: 
  /post/linux-create-rpm-package.html
  初始宏定义: 在定义文件的安装路径时，通常会使用类似 ```%_sharedstatedir``` 的宏，这些宏一般会在 ```/usr/lib/rpm/macros``` 中定义，当然部分会同时在不同平台上覆盖配置，可以直接 ```grep``` 查看。

  在安装时，可以通过 ```%files``` 段指定需要安装的目录、文件、属组、权限等，不过需要先在 ```%
  
%install
rm -rf %{buildroot}
%{__install} -Dp -m0755 contrib/init.d-uagent %{buildroot}%{_initrddir}/uagent
%{__install} -d %{buildroot}%{_sysconfdir}/uagent.d/

%files
%defattr(-, root, root, -)
%doc ChangeLog README
%attr(755, root, root) %{_bindir}/replace
%config(noreplace) %{_sysconfdir}/uagent.conf
%dir %attr(755, monitor, monitor) %{_sharedstatedir}/uagent

/post/mysql-introduce.html
  如果没有手动初始化数据直接启动，那么会将root密码打印到日志中，然后可以直接登陆；也可以修改配置文件跳过认证，也就是添加 skip-grant-tables 选项。

----- 不建议使用如下方式修改密码
> SET PASSWORD=PASSWORD('Root1234@');
> FLUSH PRIVILEGES;

----- 可以使用如下方式修改，且不需要FLUSH，建议时候后者
SET password='Root1234@';
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';

### 校验

可以通过 ```--verify``` 或者 ```-V``` 对参数进行校验，正常不会显示任何信息，可以通过 ```--verbose``` 或者 ```-v``` 显示每个文件的信息。文件丢失时则显示 ```missing```，属性方面的修改内容如下：

SM5DLUGT c filename
属性：
  S: 文件大小;
  M: 权限;
  5: MD5检查和;
  D: 主从设备号;
  L: 符号连接;
  U: 属主;
  G: 属组;
  T: 最后修改时间。
类型：
  c: 配置文件；
  d: 文档文件。


http://blog.csdn.net/liuintermilan/article/details/6283705



getaddrinfo()
http://www.cnblogs.com/cxz2009/archive/2010/11/19/1881693.html

http://leaver.me/2015/09/03/kafka%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B/
Kafka介绍   http://blog.csdn.net/suifeng3051/article/details/48053965
ZeroCPY  https://www.ibm.com/developerworks/linux/library/j-zerocopy/
Kafka failover 机制详解 http://www.cnblogs.com/fxjwind/p/4972244.html
很多不错的图片 http://zqhxuyuan.github.io/2016/01/13/2016-01-13-Kafka-Picture/

Consumer Group
  多个consumer可以组成一个组，每个消息只能被组中的一个consumer消费，如果想一个消息可以被多个consumer消费的话，那么这些consumer必须在不同的组。
消息状态
  消息的状态被保存在consumer中，broker不会关心哪个消息被消费了被谁消费了，只记录一个offset值（指向partition中下一个要被消费的消息位置），这就意味着如果consumer处理不好的话，broker上的一个消息可能会被消费多次。
    消息有效期：Kafka会长久保留其中的消息，以便consumer可以多次消费，当然其中很多细节是可配置的。
    批量发送：Kafka支持以消息集合为单位进行批量发送，以提高push效率。
    push-and-pull : Kafka中的Producer和consumer采用的是push-and-pull模式，即Producer只管向broker push消息，consumer只管从broker pull消息，两者对消息的生产和消费是异步的。
    负载均衡方面： Kafka提供了一个 metadata API来管理broker之间的负载（对Kafka0.8.x而言，对于0.7.x主要靠zookeeper来实现负载均衡）。
    同步异步：Producer采用异步push方式，极大提高Kafka系统的吞吐率（可以通过参数控制是采用同步还是异步方式）。
Partition (分区)
  一个Topic下可以有多个Partition，Kafka的broker端支持消息分区，Producer可以决定把消息发到哪个分区，在一个分区中消息的顺序就是Producer发送消息的顺序。

librdkafka 用 mklove 编译。

代码运行流程如下

rd_kafka_conf_set()
  设置全局配置，配置文件中通过Property配置
rd_kafka_topic_conf_set()
  设置topic配置
rd_kafka_brokers_add()
  设置broker地址，也就是bootstrap broker，启动向broker发送消息的线程
rd_kafka_new()
  将上述的conf作为参数，启动kafka主线程，也就是rd_kafka_thread_main()函数

rd_kafka_topic_new建topic

rd_kafka_produce使用本函数发送消息

rd_kafka_poll调用回调函数

还是看发送一条消息的过程

#define HAVE_LIBRDKAFKA_LOG_CB 1
#undef HAVE_LIBRDKAFKA_LOGGER

新版本使用 rd_kafka_conf_set_log_cb() 替换了 rd_kafka_set_logger() 接口。

简单来说，应用线程向队列扔消息，librdkafka启动的线程负责从队列里取消息并向kafka broker发送消息。

src/rdkafka_transport.c 调用操作系统的 poll() 接口。

cnd_signal()
pthread_cond_signal()


rd_kafka_replyq_enq()
 |-rd_kafka_q_enq()

发送消息
rd_kafka_produce()
 |-rd_kafka_msg_new()
   |-rd_kafka_msg_new0() 创建消息并初始化，包括消息长度、标记位、timeout等内容
   |-rd_kafka_msg_partitioner() 分区并添加到队列
   | |-rd_kafka_toppar_get() 获取partition
   | |-rd_kafka_toppar_enq_msg() 加入队列，等待broker线程取出
   |   |-rd_kafka_msgq_enq() 真正添加到topic partition队列中，注意需要进行加锁
   |     |-TAILQ_INSERT_TAIL() 添加到队列末尾
   |-rd_kafka_msg_destroy()
rd_kafka_broker_add() 

rd_kafka_new() 创建新的实例，会返回rd_kafka_t结构体
 |-rd_kafka_cgrp_new() 如果是消费者会创建分组
 |-rd_kafka_thread_main() 为生产者、消费者启动单独线程处理 rd_kafka_thread_main()
 | |-rd_snprintf() 将线程名称设置为main
 | |-rd_kafka_q_serve() 从rk_ops队列中读取消息
 |   |-rd_kafka_op_handle() 处理完成后调用回调函数
 |-rd_kafka_broker_add() 创建内部broker线程，也就是 rd_kafka_broker_thread_main()

rd_kafka_broker_thread_main()
 |-rd_kafka_broker_terminating() 如果没有关闭
 |-rd_kafka_broker_connect() 在初始化以及STATE_DOWN时，不断重试链接
 | |-rd_kafka_broker_resolve()
 | | |-rd_getaddrinfo()
 | |-rd_kafka_transport_connect() 设置网络异步通讯
 | | |-rd_fd_set_nonblocking()
 | | |-rd_kafka_transport_poll_set()
 | |-rd_kafka_broker_set_state()
 |
 |-rd_kafka_broker_producer_serve() 处于STATE_UP时根据不同的类型调用不同函数
 | |-rd_kafka_broker_terminating()
 | |-rd_kafka_toppar_producer_serve() 会通过TAILQ_FOREACH()从队列中读取，然后调用该函数
     |-rd_kafka_msgq_concat() 如果有需要发送的消息，则直接将消息从rktp->rktp_msgq移动到rktp->rktp_xmit_msgq，并清空前者
     |-rd_kafka_msgq_age_scan()
  |-rd_kafka_broker_produce_toppar()
    |-rd_kafka_buf_new() 新建发送缓冲区
 | |-rd_kafka_broker_serve() 从队列中读取消息，并将数据发送到网络
 |   |-rd_kafka_q_pop() 会返回具体的操作类型
 |   | |-rd_kafka_q_pop_serve()
 |   |-rd_kafka_broker_op_serve() 根据不同的操作类型调用不同函数接口
 |   |
 |   |-rd_kafka_transport_io_serve()
 |     |-rd_kafka_transport_poll() 调用操作系统的poll()系统调用接口
 |     |-rd_kafka_transport_poll_clear()
 |     |-rd_kafka_transport_io_event() 处理返回的IO事件
 |       |-rd_kafka_send() 例如发送
 |       |-rd_kafka_recv() 以及接收
 |-rd_kafka_broker_consumer_serve()

这也就意味着应用程序将消息发送给 rdkafka，然后 rdkafka 会直接将消息保存到队列中，并由其它线程异步发送给 broker 。

http://blog.csdn.net/auwzb/article/details/9665729
http://www.cnblogs.com/xhcqwl/p/3905412.html
http://codingeek.me/2017/04/16/librdkafka%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/


在调用 ```rd_kafka_new()``` 函数时，会根据入参是消费者还是生产者返回 ```rd_kafka_t``` 对象，此时会同时创建一个主处理线程，也就是 ```rd_kafka_thread_main()``` 。

在主线程中会从 ```rd_kafka_t.rk_ops``` 队列中逐一读取消息，然后根据不同的操作类型 (通过rd_kafka_op_type_t定义) 分别进行处理，其中操作通过 rd_kafka_op_t 定义，操作类型对应了其中的 rko_type 成员，操作执行完后调用 rd_kafka_op_handle() 回调。

Kafka 中可以将 Topic 从物理上划分成一个或多个分区 (Partition)，每个分区在物理上对应一个文件夹，以 "TopicName_PartitionIndex" 的命名方式命名，该文件夹下存储这个分区的所有消息 (.log) 和索引文件 (.index)，这使得 Kafka 的吞吐率可以水平扩展。

在通过命令行创建 Topic 时，可以使用 --partitions <numPartitions> 指定分区数；也可以在 server.properties 配置文件中配置参数 num.partitions 来指定默认的分区数。需要注意的是，在为 Topic 创建分区时，分区数最好是 broker 数量的整数倍。

可以通过 rd_kafka_broker_add() 创建 broker 线程，有如下三种类型：
    RD_KAFKA_CONFIGURED
    根据用户配置，生成的broker线程
    RD_KAFKA_LEARNED
    内部使用的broker线程，主要针对Client Group使用
    RD_KAFKA_INTERNAL
    内部使用的broker线程
#define unlikely(x) __builtin_expect((x),0)
http://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html


←
-->

{% highlight text %}
{% endhighlight %}

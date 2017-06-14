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



Page Cache
https://www.thomas-krenn.com/en/wiki/Linux_Page_Cache_Basics

Page Cache, the Affair Between Memory and Files

http://duartes.org/gustavo/blog/post/page-cache-the-affair-between-memory-and-files/

https://www.quora.com/What-is-the-major-difference-between-the-buffer-cache-and-the-page-cache

从free到page cache

http://www.cnblogs.com/hustcat/archive/2011/10/27/2226995.html



getaddrinfo()
http://www.cnblogs.com/cxz2009/archive/2010/11/19/1881693.html


#define unlikely(x) __builtin_expect((x),0)
http://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html


libcurl使用
http://www.cnblogs.com/moodlxs/archive/2012/10/15/2724318.html


FIXME:
   /post/collectd-source-code.html
   cf_include_all() 在解析完上述配置文件后查看是否有Include选项
   /post/collectd-source-code.html
 plugin_write() 所有插件都写入失败时返回-1，否则返回0
   /post/linux-package.html
   有红色输出，语法问题。
   AC_CONFIG_FILES 主要是通过 ```*.in``` 模板生成响应的文件。

   关于 autotools 的简单处理流程可以参考 [automake](http://www.gnu.org/software/automake/manual/automake.html) 中的 ```Setup Explained``` 内容。

http://people.ds.cam.ac.uk/jw35/docs/rpm_config.html
%config(noreplace)
新安装时如果配置文件存在则会将原文件保存为 *.rpmsave ，升级时则不会覆盖原文件，直接将包中的文件命名为 *.rpmnew 。


### simple & matching=>默认行为

在 git 全局配置中，有个 push.default 属性决定了 git push 操作的默认行为，在 2.0 之前，默认为 'matching'，2.0 之后则被更改为了 'simple'。除此之外，还有如下的几个配置项：

* nothing<br>push操作无效，除非显式指定远程分支，例如 git push origin develop。
* current<br>push当前分支到远程同名分支，如果远程同名分支不存在则自动创建同名分支。
* upstream<br>push当前分支到它的upstream分支上，常用于从本地分支push/pull到同一远程仓库的情景，这种模式叫做central workflow。
* simple<br>simple和upstream是相似的，只有一点不同，simple必须保证本地分支和它的远程upstream分支同名，否则会拒绝push操作。
* matching<br>push所有本地和远程两端都存在的同名分支。

初次提交本地分支，例如 git push origin develop 操作，并不会定义当前本地分支的upstream分支，可以通过git push --set-upstream origin develop，关联本地develop分支的upstream分支，另一个更为简洁的方式是初次push时，加入-u参数，例如git push -u origin develop，这个操作在push的同时会指定当前分支的upstream。

 git branch --set-upstream-to=origin/<branch> new1
http://blog.angular.in/git-pushmo-ren-fen-zhi/
http://ybin.cc/git/git-default-push-option-explanation/
https://orga.cat/posts/most-useful-git-commands
https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%E4%B8%8E%E6%B8%85%E7%90%86#_git_stashing
https://lwn.net/Articles/584225/
https://en.wikipedia.org/wiki/Stack_buffer_overflow#Stack_canaries
https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html
https://outflux.net/blog/archives/2014/01/27/fstack-protector-strong/
-pipe
  从源码生成可执行文件一般需要四个步骤，并且还会产生中间文件，该参数用于配置实用PIPE，一些平台会失败，不过 GNU 不受影响。
-fexceptions
  打开异常处理，该选项会生成必要的代码来处理异常的抛出和捕获，对于 C++ 等会触发异常的语言来说，默认都会指定该选项。所生成的代码不会造成性能损失，但会造成尺寸上的损失。因此，如果想要编译不使用异常的 C++ 代码，可能需要指定选项 -fno-exceptions 。
-Wall -Werror -O2 -g --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1  -m64 -mtune=generic -DLT_LAZY_OR_NOW="RTLD_LAZY|RTLD_GLOBAL"
http://www.cis.syr.edu/~wedu/seed/Labs/Documentation/Linux/How_Linux_Capability_Works.pdf
http://blog.siphos.be/2013/05/overview-of-linux-capabilities-part-1/




关于Linux内核很不错的介绍
http://duartes.org/gustavo/blog/






chcon 用于修改文件的 SELinux 安全上下文，或者说是修改安全策略。

进程所属的上下文类型称为域，而文件所属的上下文类型称为类型，在 SELinux 开启后，进程只能操作域类型与上下文类型一样的文件。

在 CentOS 中，semanage 命令默认没有安装，可以通过 ```yum install policycoreutils-python``` 安装。

----- 查看哪些服务受SELinux管理，也就是布尔值；包括了如何设置，-P永久生效
# getsebool -a
# setsebool samba_enable_home_dirs=1
# setsebool samba_enable_home_dirs=on
----- 恢复默认上下文
# restorecon /var/www/html
----- 默认上下文类型？保存在那个文件中
# semanage fcontext -l
----- 用户映射关系
# semanage user -l
----- 查看受SELinux控制的端口，以及将指定端口添加到规则中
# semanage port -l
# semanage port -a -t http_port_t -p tcp 8060

chcon [选项]... 环境 文件...
chcon [选项]... [-u 用户] [-r 角色] [-l 范围] [-t 类型] 文件...
chcon [选项]... --reference=参考文件 文件...

二、chcon命令参数：
参数名 描述
-u 用户
--user=用户 设置指定用户的目标安全环境；
 -r 角色
--role=角色 设置指定角色的目标安全环境；
-t 类型
--type=类型 设置指定类型的目标安全环境；
-l 范围
--range=范围 设置指定范围的目标安全环境；
-v
--verbose 处理的所有文件时显示诊断信息；

-R/--recursive
  递归处理目录、文件；


-h
--no-dereference 影响符号连接而非引用的文件；
--reference=file 使用指定参考文件file的安全环境，而非指定值；
-H  如果一个命令行参数是一个目录的符号链接，则遍历它；
-L 遍历每一个符号连接指向的目录；
-P 不遍历任何符号链接（默认）；
--help 显示此帮助信息并退出；
--version 显示版本信息并退出；

三、chcon用法演示：
1、mysql：让selinux

允许mysqld做为数据目录访问“/storage/mysql”：
-----
# chcon -R -t mysqld_db_t /storage/mysql
2、ftp：将共享目录开放给匿名用户：
[root@aiezu.com ~]# chcon -R -t public_content_t /storage/ftp/
#允许匿名用户上传文件：
[root@aiezu.com ~]# chcon -R -t public_content_rw_t /storage/ftp
[root@aiezu.com ~]# setsebool -P allow_ftpd_anon_write=1

3、为网站目录开放httpd访问权限：
chcon -R -t httpd_sys_content_t /storage/web

https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/chap-Security-Enhanced_Linux-SELinux_Contexts.html
https://debian-handbook.info/browse/zh-CN/stable/sect.selinux.html
https://wiki.centos.org/zh/HowTos/SELinux
http://blog.siphos.be/2013/05/overview-of-linux-capabilities-part-1/
http://www.cis.syr.edu/~wedu/seed/Labs/Documentation/Linux/How_Linux_Capability_Works.pdf


编程时常有 Client 和 Server 需要各自得到对方 IP 和 Port 的需求，此时就可以通过 getsockname() 和 getpeername() 获取。

python -c '
import sys
import socket
s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("www.huawei.com", 80))
print s.getsockname()
s.close()'

FIXME:
  rsync
  --exclude "checkout"    某个目录
  --exclude "filename*"   某类文件
  --exclude-from=sync-exclude.list  通过文件指定要忽略的文件
注意，指定时使用的是源地址的相对路径。

Python判断IP有效性
https://gist.github.com/youngsterxyf/5088954

/post/centos-config-from-scratch.html
# 查询未安装软件包的依赖关系
rpm -qRp vim-common-6.3.046-2.el4.1.x86_64.rpm
# 查询已安装软件包的依赖关系
rpm -qR vim-common-6.3.046-2.el4.1

gpg签名
/etc/pki/rpm-gpg/RPM*

rpm 安装时可能会报 NOKEY 的错误信息 --nogpgcheck nosignature

iptables -I INPUT -s 192.30.0.0/16 -p tcp --dport 21005 -j ACCEPT
/post/linux-commands-text.html
  如果其中的部分参数需要动态获取，而 ```''``` 则会原样输出字符内容，那么可以通过 ```echo "'$(hostname)'" | xargs sed filename -e``` 这种方式执行。
/post/network-setting.html

ip route list

route -n
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.145.12.254  0.0.0.0         UG    0      0        0 eth0
100.125.0.0     0.0.0.0         255.255.248.0   U     0      0        0 eth1

Destination、Gateway、Genmask，目标地址、网管、掩码

第一行表示主机所在网络的地址为192.168.120.0，若数据传送目标是在本局域网内通信，则可直接通过eth0转发数据包;

第四行表示数据传送目的是访问Internet，则由接口eth0，将数据包发送到网关192.168.120.240

Flags 为路由标志，标记当前网络节点的状态，各个标志简述如下：
  U Up，此路由当前为启动状态
  G Gateway，此网关为一路由器
  ! Reject Route，表示此路由当前为关闭状态，常用于抵挡不安全规则
  H Host，表示此网关为一主机
  R Reinstate Route，使用动态路由重新初始化的路由
  D Dynamically,此路由是动态性地写入
  M Modified，此路由是由路由守护程序或导向器动态修改
Metric

metric Metric 为路由指定一个整数成本值标（从 1 至 9999），当在路由表(与转发的数据包目标地址最匹配)的多个路由中进行选择时可以使用。

http://www.cnblogs.com/vamei/archive/2012/10/07/2713023.html

----- 增加一条到达244.0.0.0的路由
# route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0
----- 增加一条屏蔽的路由，目的地址为224.x.x.x将被拒绝
# route add -net 224.0.0.0 netmask 240.0.0.0 reject
----- 删除路由记录
# route del -net 224.0.0.0 netmask 240.0.0.0
# route del -net 224.0.0.0 netmask 240.0.0.0 reject
----- 删除和添加设置默认网关
# route del default gw 192.168.120.240
# route add default gw 192.168.120.240

Routing Cache 也被称为 Forwarding Information Base, FIB ，通过 Hash 表保存了最近使用的路由信息，如果在 Cache 找到了对应的路由信息，那么会直接使用该规则进行转发，而不再查找路由表。

注意，Routing Cache 和 Route Table 是不相关的，所以在修改路由表之后最后手动清空下 Cache ，可以通过 ```/proc/net/rt_cache``` 文件查看或者使用如下命令：

ip route show cache
ip route flush cache
ip route get

关于 IP 命令的详细可以查看 [IP Route Management](http://linux-ip.net/html/tools-ip-route.html) 。

http://linux-ip.net/html/routing-selection.html

一般路由匹配的流程是：
A. 先匹配掩码，掩码最精确匹配的路由优先 (longest prefix match)；
B. 如果有多条路由，则匹配 [管理距离](https://en.wikipedia.org/wiki/Administrative_distance)，管理距离小的路由优先；
C. 如果管理距离相同，在匹配度量值 (lowest metric)，度量值小的优先
D. 如果度量值相同，则选择负载均衡，具体的方式看采用哪种路由协议和相关的配置了。

安全渗透工具集
https://wizardforcel.gitbooks.io/daxueba-kali-linux-tutorial/content/2.html

git stash save              保存，不带子命令的默认值
git stash apply stash@{0}   默认应用第一个，注意，此时不会删除保存的stash
git stash drop stash@{0}    手动删除
git stash list [<options>]
git stash show [<stash>]
git stash ( pop | apply ) [--index] [-q|--quiet] [<stash>]
git stash branch <branchname> [<stash>]
git stash [push [-p|--patch] [-k|--[no-]keep-index] [-q|--quiet]
      [-u|--include-untracked] [-a|--all] [-m|--message <message>]]
      [--] [<pathspec>…]]
git stash clear
git stash create [<message>]
git stash store [-m|--message <message>] [-q|--quiet] <commit>

在引用时，可以使用类似如下的方式 ```stash@{0}```、```stash@{2.hours.ago}``` 。

hostname获取方式，在启动时通过 1) global_option_get() 配置文件获取；2) gethostname()；3) getaddrinfo()。

#include <unistd.h>
int gethostname(char *name, size_t len);
  返回本地主机的标准主机名；正常返回 0 否则返回 -1，错误码保存在 errno 中。

#include <netdb.h>
#include <sys/socket.h>
struct hostent *gethostbyname(const char *name);
  用域名或主机名获取IP地址，注意只支持IPv4；正常返回一个 struct hostent 结构，否则返回 NULL。

#include<netdb.h>
int getaddrinfo(const char *hostname, const char *service, const struct addrinfo *hints, struct addrinfo **result);
  hostname: 一个主机名或者地址串，IPv4的点分十进制串或者IPv6的16进制串；
  service : 服务名可以是十进制的端口号，也可以是已定义的服务名称，如ftp、http等；
  hints   : 可以为空，用于指定返回的类型信息，例如，服务支持 TCP/UDP 那么，可以设置 ai_socktype 为 SOCK_DGRAM 只返回 UDP 信息；
  result  : 返回的结果。
  返回 0 成功。

struct addrinfo {
    int              ai_flags;
    int              ai_family;
    int              ai_socktype;
    int              ai_protocol;
    socklen_t        ai_addrlen;
    struct sockaddr *ai_addr;        // IP地址，需要通过inet_ntop()转换为IP字符串
    char            *ai_canonname;   // 返回的主机名
    struct addrinfo *ai_next;
};
http://blog.csdn.net/a_ran/article/details/41871437


const char *inet_ntop(int af, const void *src, char *dst, socklen_t cnt);
  将类型为af的网络地址结构src，转换成主机序的字符串形式，存放在长度为cnt的字符串中。返回指向dst的一个指针。如果函数调用错误，返回值是NULL。

通常 capabilities 是和 SELinux 配合使用的，以往，如果要运行一个需要 root 权限的程序，那么需要保证有运行权限，且是 root 用户；通过 capabilities 就可以只赋予程序所需要的权限。

以 ping 命令为例，因为需要发送 raw 格式数据，部分发行版本使用了 setuid + root，实际上只需要赋予 CAP_NET_RAW 权限，然后去除 setuid 即可。


----- 直接复制一个ping命令，然后进行测试
# cp ping anotherping
# chcon -t ping_exec_t anotherping
$ ping -c 1 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.057 ms
$ anotherping -c 1 127.0.0.1
ping: icmp open socket: Operation not permitted

----- 新增CAP_NET_RAW权限，然后用非root用户重新测试
# setcap cap_net_raw+ep anotherping
$ anotherping -c 1 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.054 ms

user
role
type
level








←
-->

{% highlight text %}
{% endhighlight %}

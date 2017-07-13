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




https://lwn.net/Articles/584225/
https://en.wikipedia.org/wiki/Stack_buffer_overflow#Stack_canaries
https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html
https://outflux.net/blog/archives/2014/01/27/fstack-protector-strong/
-pipe
  从源码生成可执行文件一般需要四个步骤，并且还会产生中间文件，该参数用于配置实用PIPE，一些平台会失败，不过 GNU 不受影响。
-fexceptions
  打开异常处理，该选项会生成必要的代码来处理异常的抛出和捕获，对于 C++ 等会触发异常的语言来说，默认都会指定该选项。所生成的代码不会造成性能损失，但会造成尺寸上的损失。因此，如果想要编译不使用异常的 C++ 代码，可能需要指定选项 -fno-exceptions 。
-Wall -Werror -O2 -g --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1  -m64 -mtune=generic -DLT_LAZY_OR_NOW="RTLD_LAZY|RTLD_GLOBAL"




关于Linux内核很不错的介绍
http://duartes.org/gustavo/blog/






编程时常有 Client 和 Server 需要各自得到对方 IP 和 Port 的需求，此时就可以通过 getsockname() 和 getpeername() 获取。

python -c '
import sys
import socket
s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("www.huawei.com", 80))
print s.getsockname()
s.close()'


Python判断IP有效性
https://gist.github.com/youngsterxyf/5088954



安全渗透工具集
https://wizardforcel.gitbooks.io/daxueba-kali-linux-tutorial/content/2.html

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



对于不信任的组件建议使用后者，因为 ldd 可能会加载后显示依赖的库，从而导致安全问题。

----- 查看依赖的库
$ ldd /usr/bin/ssh
$ objdump -p /usr/bin/ssh | grep NEEDED

----- 运行程序加载的库
# pldd $(pidof mysqld)


# pldd $(pidof uagent)


VIRT, Virtual Memory Size @
  该任务的总的虚拟内存，包括了 code、data、shared libraries、换出到磁盘的页、已经映射但是没有使用的页。
USED, Memory in Use @
  包括了已使用的物理内存 RES ，以及换出到磁盘的内存 SWAP。
%MEM, Memory Usage(RES) @
  当前任务使用的内存与整个物理内存的占比。
CODE, Code Size
  可执行代码占用的物理内存数，也被称为 Text Resident Set, TRS。
DATA, Data+Stack Size
  除了代码之外的物理内存占用数，也就是 Data Resident Set, DRS 。
RES, Resident Memory Size @
  驻留在物理内存中的使用量。
SHR, Shared Memory Size @
  包括了共享内存以及共享库的数据。

SWAP, Swapped Size
  换出到磁盘的内存。
nMaj, nMin, nDRT


RES = CODE + DATA???? DATA太大了，为什么

====== ps
DRS, Data Resident Set <=> top(DATA) !!!
  除了代码之外的物理内存占用数。
RSS, Resident Set Size <=> top(RES)
  物理内存使用数。
TRS, Text Resident Set <=> top(CODE) !!!
  代码在内存中的占用数。
VSZ, Virtual Memory Size <=> top(VIRT) <=> pmap -d(mapped)
  虚拟内存的大小。

RES(top) 和 RSS(ps) 实际上读取的是 /proc/$(pidof process)/stat 或者 /proc/$(pidof process)/status statm。
pmap -d $(pidof uagent)
pmap -x $(pidof uagent)
ps -o pid,pmem,drs,trs,rss,vsz Hp `pidof uagent`

另外，cgtop 中显示的内存与什么相关？？？？
ps(TRS) 和 top(CODE) 的值不相同。

http://blog.csdn.net/u011547375/article/details/9851455
https://stackoverflow.com/questions/7594548/res-code-data-in-the-output-information-of-the-top-command-why
https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/36669.pdf
https://landley.net/kdocs/ols/2010/ols2010-pages-245-254.pdf
LDD 跟 PMAP 加载的库不同？

awk 'BEGIN{sum=0};{if($3~/x/) {sum+=$2}};END{print sum}' /tmp/1

procs_refresh

Top用于查看Linux系统下进程信息，有时候需要选择显示那些列，以及按照某一列进行排序。查询整理如下：


top 除了默认的列之外，可以选择需要显示的列，操作如下：

----- 选择需要显示的列
1) 按 f 键进入选择界面；2) 方向键选择需要的列；3) 通过空格选择需要显示的列。

列显示位置调整：
执行top命令后，按 o 键，选择要调整位置的列（如K:CUP Usageage），按动一下大写K则显示位置往上调整，按动一下小写K则显示位置往下调整。

列排序：
执行top命令后，按 shift + f（小写），进入选择排序列页面，再按要排序的列的代表字母即可；

systemctl set-property --runtime uagent.service CPUQuota=5% MemoryLimit=30M

关于资源配置的选项可以通过 ```man 5 systemd.resource-control``` 方式查看，默认是没有开启审计的，所以通过 ```systemd-cgtop``` 没有显示具体的资源。

很多相关的内核文档链接
https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html

注册信号处理函数

setsid()
pidfile_create()

https://www.ibm.com/support/knowledgecenter/zh/ssw_aix_61/com.ibm.aix.genprogc/ie_prog_4lex_yacc.htm

flex 通过 yylval 将数据传递给 yacc；如果在 yacc 中使用了 ```%union``` ，那么各个条件的目的变量使用 yyval 。

https://www.howtoforge.com/storing-files-directories-in-memory-with-tmpfs



meminfo详解
https://lwn.net/Articles/28345/

ps的SIZE以及RSS不含部分的内存统计，所以要比pmap -d统计的RSS小。
The SIZE and RSS fields don't count some parts of a process including the page tables, kernel stack, struct thread_info
https://techtalk.intersec.com/2013/07/memory-part-2-understanding-process-memory/
http://tldp.org/LDP/tlk/mm/memory.html
http://tldp.org/LDP/khg/HyperNews/get/memory/linuxmm.html
https://lwn.net/Articles/230975/
https://gist.github.com/CMCDragonkai/10ab53654b2aa6ce55c11cfc5b2432a4
https://yq.aliyun.com/ziliao/75375
http://elinux.org/Runtime_Memory_Measurement
https://access.redhat.com/security/vulnerabilities/stackguard
http://events.linuxfoundation.org/sites/events/files/slides/elc_2016_mem_0.pdf
http://blog.csdn.net/lijzheng/article/details/23618365
https://yq.aliyun.com/articles/54405
https://stackoverflow.com/questions/31328349/stack-memory-management-in-linux
/post/mysql-parser.html
yyset_in() 设置入口


pmap -d $(pidof uagent)
pmap -x $(pidof uagent)

top -Hp $(pidof uagent)
ps -o pid,pmem,drs,trs,rss,vsz Hp `pidof uagent`
/proc/$(pidof uagent)/stat
/proc/$(pidof uagent)/status
/proc/$(pidof uagent)/maps
VmRSS、VmSize

$ ps aux|grep /usr/bin/X|grep -v grep | awk '{print $2}'   # 得出X server 的 pid   ...
1076
$ cat /proc/1076/stat | awk '{print $23 / 1024}'
139012
$ cat /proc/1076/status | grep -i vmsize
VmSize:      106516 kB
VmSize = memory + memory-mapped hardware (e.g. video card memory).

kmap 是用来建立映射的，映射后返回了被映射的高端内存在内核的线性地址
https://www.zhihu.com/question/30338816
http://blog.csdn.net/gatieme/article/details/52705142
http://www.cnblogs.com/zhiliao112/p/4251221.html
http://way4ever.com/?p=236
awk统计Linux最常用命令
http://www.ha97.com/3980.html
awk使用技巧
http://blog.csdn.net/ultrani/article/details/6750434
http://blog.csdn.net/u011204847/article/details/51205031 *****
http://ustb80.blog.51cto.com/6139482/1051310


关于smaps的详细介绍
https://jameshunt.us/writings/smaps.html
$ cat /proc/self/smaps  相比maps显示更详细信息
$ cat /proc/self/maps
address                  perms   offset   dev   inode       pathname
7f571af7a000-7f571af7d000 ---p 00000000 00:00 0
7f571af7d000-7f571b080000 rw-p 00000000 00:00 0             [stack:4714]
7f571b0ac000-7f571b0ad000 r--p 00021000 08:01 1838227       /usr/lib/ld-2.21.so
7ffe49dbd000-7ffe49dbf000 r-xp 00000000 00:00 0             [vdso]

列说明:
    starting address - ending address
    permissions
        r : read
        w : write
        x : execute
        s : shared
        p : private (copy on write)
    offset   : 如果不是file，则为0；
    device   : 如果是file，则是file所在device的major and monior device number，否则为00:00；
    inode    : 如果是file，则是file的inode number，否则为0；
    pathname : 有几种情况；
        file absolute path
        [stack]         the stack of the main process
        [stack:1001]    the stack of the thread with tid 1001
        [heap]
        [vdso] - virtual dynamic shared object, the kernel system call handler
        空白 -通常都是mmap创建的，用于其他一些用途的，比如共享内存


df -h
ls /dev/XXX -alh
echo $((0x0a))

print "Backed by file:\n";
print "  RO data                   r--  $mapped_rodata\n";
print "  Unreadable                ---  $mapped_unreadable\n"; 共享库同时存在？？？？
print "  Unknown                        $mapped_unknown\n";
print "Anonymous:\n";
print "  Writable code (stack)     rwx  $writable_code\n";
print "  Data (malloc, mmap)       rw-  $data\n";
print "  RO data                   r--  $rodata\n";
print "  Unreadable                ---  $unreadable\n";
print "  Unknown                        $unbacked_unknown\n";

16进制求和，都是16进制
awk --non-decimal-data  '{sum=($1 + $2); printf("0x%x %s\n", sum,$3)}'
strtonum("0x" $1)
echo $(( 16#a36b ))
echo "obase=2;256"|bc ibase base


print "Backed by file:\n";
print "  Unreadable                ---  $mapped_unreadable\n"; 共享库同时存在？？？？
print "  Unknown                        $mapped_unknown\n";
print "Anonymous:\n";
print "  Unreadable                ---  $unreadable\n";
print "  Unknown                        $unbacked_unknown\n";

代码
r-x 代码，包括程序(File)、共享库(File)、vdso(2Pages)、vsyscall(1Page)
rwx 没有，Backed by file: Write/Exec (jump tables); Anonymous: Writable code (stack)
r-- 程序中的只读数据，如字符串，包括程序(File)、共享库(File)
rw- 可读写变量，如全局变量；包括程序(File)、共享库(File)、stack、heap、匿名映射

静态数据、全局变量将保存在 ELF 的 .data 段中。
与smaps相关，以及一些实例
https://jameshunt.us/writings/smaps.html


各共享库的代码段，存放着二进制可执行的机器指令，是由kernel把该库ELF文件的代码段map到虚存空间；
各共享库的数据段，存放着程序执行所需的全局变量，是由kernel把ELF文件的数据段map到虚存空间；

用户代码段，存放着二进制形式的可执行的机器指令，是由kernel把ELF文件的代码段map到虚存空间；
用户数据段之上是代码段，存放着程序执行所需的全局变量，是由kernel把ELF文件的数据段map到虚存空间；

用户数据段之下是堆(heap)，当且仅当malloc调用时存在，是由kernel把匿名内存map到虚存空间，堆则在程序中没有调用malloc的情况下不存在；
用户数据段之下是栈(stack)，作为进程的临时数据区，是由kernel把匿名内存map到虚存空间，栈空间的增长方向是从高地址到低地址。

https://wiki.wxwidgets.org/Valgrind_Suppression_File_Howto


另外，可以通过 ldd 查看对应的映射地址，在实际映射到物理内存时，会添加随机的变量，不过如上的各个共享库的地址是相同的。

可以通过 echo $(( 0x00007f194de48000 - 0x00007f194dc2c000)) 计算差值。


maps 文件对应了内核中的 show_map()

show_map()
 |-show_map_vma()

address                  perms   offset   dev   inode       pathname

http://duartes.org/gustavo/blog/post/how-the-kernel-manages-your-memory/


主要是anon中的rw属性导致
cat /proc/$(pidof uagent)/maps | grep stack | wc -l



Clean_pages 自从映射之后没有被修改的页；
Dirty_pages 反之；
RSS 包括了共享以及私有，Shared_Clean+Shared_Dirty、Private_Clean+Private_Dirty
PSS (Proportional set size) 包括了所有的私有页 (Private Pages) 以及共享页的平均值。例如，一个进程有100K的私有页，与一个进程有500K的共享页，与四个进程有500K的共享页，那么 PSS=100K+(500K/2)+(500K/5)=450K
USS (Unique set size) 私有页的和。

awk -f test.awk /proc/$(pidof uagent)/maps
#! /bin/awk -f
BEGIN {
    mapped_executable    = 0
    mapped_wrexec        = 0
    mapped_rodata        = 0
    mapped_rwdata        = 0
    mapped_unreadable    = 0
    mapped_unknown       = 0
    writable_code        = 0
    data                 = 0
    rodata               = 0
    unreadable           = 0
    vdso                 = 0
    unbacked_unknown     = 0
}

{
    split($1, addr, "-")
    pages = (strtonum("0x" addr[2]) - strtonum("0x" addr[1]))/4096
    if ( $4 == "00:00") {
        if      ( $2 ~ /rwx/ ) {     writable_code += pages }
        else if ( $2 ~ /rw-/ ) {              data += pages }
        else if ( $2 ~ /r-x/ ) {              vdso += pages }
        else if ( $2 ~ /r--/ ) {            rodata += pages }
        else if ( $2 ~ /---/ ) {        unreadable += pages }
        else                   {  unbacked_unknown += pages }
    } else {
        if      ( $2 ~ /rwx/ ) {     mapped_wrexec += pages }
        else if ( $2 ~ /rw-/ ) {     mapped_rwdata += pages }
        else if ( $2 ~ /r-x/ ) { mapped_executable += pages }
        else if ( $2 ~ /r--/ ) {     mapped_rodata += pages }
        else if ( $2 ~ /---/ ) { mapped_unreadable += pages }
        else                   {    mapped_unknown += pages }
    }
}
END {
    printf ("Backed by file:\n")
    printf ("  Write/Exec (jump tables)  rwx  %d\n", mapped_wrexec)
    printf ("  Data                      rw-  %d\n", mapped_rwdata)
    printf ("  Executable                r-x  %d\n", mapped_executable)
    printf ("  RO data                   r--  %d\n", mapped_rodata)
    printf ("  Unreadable                ---  %d\n", mapped_unreadable)
    printf ("  Unknown                        %d\n", mapped_unknown)
    printf ("Anonymous:\n")
    printf ("  Writable code (stack)     rwx  %d\n", writable_code)
    printf ("  Data (malloc, mmap)       rw-  %d\n", data)
    printf ("  vdso, vsyscall            r-x  %d\n", vdso)
    printf ("  RO data                   r--  %d\n", rodata)
    printf ("  Unreadable                ---  %d\n", unreadable)
    printf ("  Unknown                        %d\n", unbacked_unknown)
}

pmap -x $(pidof uagent) > /tmp/1
awk -f test.awk /tmp/1
#! /bin/awk -f
BEGIN {
    lib_dirty_rx         = 0
    lib_dirty_rw         = 0
    lib_dirty_r          = 0
    lib_dirty_unknown    = 0
    lib_rss_rx         = 0
    lib_rss_rw         = 0
    lib_rss_r          = 0
    lib_rss_unknown    = 0

    uagent_dirty_rx      = 0
    uagent_dirty_rw      = 0
    uagent_dirty_r       = 0
    uagent_dirty_unknown = 0
    uagent_rss_rx      = 0
    uagent_rss_rw      = 0
    uagent_rss_r       = 0
    uagent_rss_unknown = 0

    anon_dirty_rw        = 0
    anon_dirty_rx        = 0
    anon_dirty_unknown   = 0
    anon_dirty_r         = 0
    anon_rss_rw        = 0
    anon_rss_rx        = 0
    anon_rss_unknown   = 0
    anon_rss_r         = 0

    count          = 0
    actual          = 0
}

$NF ~ /(^lib|^ld)/ {
    if      ( $5 ~ /r-x/ ) {
        lib_rss_rx += $3
        lib_dirty_rx += $4
    } else if ( $5 ~ /rw-/ ) {
        lib_rss_rw += $3
        lib_dirty_rw += $4
    } else if ( $5 ~ /r--/ ) {
        lib_rss_r += $3
        lib_dirty_r += $4
    } else {
        lib_rss_unknown += $3
        lib_dirty_unknown += $4
    }
    count += $3
}
$NF ~ /([a-zA-Z]+\.so$|^uagent\>)/ {
    if      ( $5 ~ /r-x/ ) {
        uagent_rss_rx += $3
        uagent_dirty_rx += $4
    } else if ( $5 ~ /rw-/ ) {
        uagent_rss_rw += $3
        uagent_dirty_rw += $4
    } else if ( $5 ~ /r--/ ) {
        uagent_rss_r += $3
        uagent_dirty_r += $4
    } else {
        uagent_rss_unknown += $3
        uagent_dirty_unknown += $4
    }
    count += $3
}
$NF ~ /^]$/ {
    if      ( $5 ~ /r-x/ ) {
        anon_rss_rx += $3
        anon_dirty_rx += $4
    } else if ( $5 ~ /rw-/ ) {
        anon_rss_rw += $3
        anon_dirty_rw += $4
    } else if ( $5 ~ /r--/ ) {
        anon_rss_r += $3
        anon_dirty_r += $4
    } else {
        anon_rss_unknown += $3
        anon_dirty_unknown += $4
    }
    count += $3
}
$1 ~ /^total\>/ {
    actual = $4
}
END {
    printf ("Libraries info:\n")
    printf (" Perm        RSS   Dirty\n")
    printf ("  r-x        %5d     %5d\n", lib_rss_rx, lib_dirty_rx)
    printf ("  rw-        %5d     %5d\n", lib_rss_rw, lib_dirty_rw)
    printf ("  r--        %5d     %5d\n", lib_rss_r,  lib_dirty_r)
    printf ("  Unknown    %5d     %5d\n", lib_rss_unknown, lib_dirty_unknown)

    printf ("Uagent info:\n")
    printf (" Perm        RSS   Dirty\n")
    printf ("  r-x        %5d     %5d\n", uagent_rss_rx,      uagent_dirty_rx)
    printf ("  rw-        %5d     %5d\n", uagent_rss_rw,      uagent_dirty_rw)
    printf ("  r--        %5d     %5d\n", uagent_rss_r,       uagent_dirty_r)
    printf ("  Unknown    %5d     %5d\n", uagent_rss_unknown, uagent_dirty_unknown)

    printf ("Anon info:\n")
    printf (" Perm        RSS   Dirty\n")
    printf ("  r-x        %5d     %5d\n", anon_rss_rx,      anon_dirty_rx)
    printf ("  rw-        %5d     %5d\n", anon_rss_rw,      anon_dirty_rw)
    printf ("  r--        %5d     %5d\n", anon_rss_r,       anon_dirty_r)
    printf ("  Unknown    %5d     %5d\n", anon_rss_unknown, anon_dirty_unknown)

    printf ("\nCount: %d  Actual: %d\n", count, actual)
}






可以通过mprotect设置内存的属性
https://linux.die.net/man/2/mprotect
Memory protection keys
https://lwn.net/Articles/643797/
Memory Protection and ASLR on Linux
https://eklitzke.org/memory-protection-and-aslr

ES Collectd插件
https://www.elastic.co/guide/en/logstash/current/plugins-codecs-collectd.html




真随机数等生成
http://www.cnblogs.com/bigship/archive/2010/04/04/1704228.html

在打印时，如果使用了 size_t 类型，那么通过 ```%d``` 打印将会打印一个告警，可以通过如下方式修改，也就是添加 ```z``` 描述。

size_t x = ...;
ssize_t y = ...;
printf("%zu\n", x);  // prints as unsigned decimal
printf("%zx\n", x);  // prints as hex
printf("%zd\n", y);  // prints as signed decimal

/proc/iomem 保存物理地址的映射情况，每行代表一个资源 (地址范围和资源名)，其中可用物理内存的资源名为 "System RAM" ，在内核中通过 insert_resource() 这个API注册到 iomem_resource 这颗资源树上。

例如，如下的内容：

01200000-0188b446 : Kernel code
0188b447-01bae6ff : Kernel data
01c33000-01dbbfff : Kernel bss

这些地址范围都是基于物理地址的，在 ```setup_arch()@arch/x86/kernel/setup.c``` 中通过如下方式注册。

max_pfn = e820_end_of_ram_pfn();
        code_resource.start = __pa_symbol(_text);
        code_resource.end = __pa_symbol(_etext)-1;
        insert_resource(&iomem_resource, &code_resource);

linux虚拟地址转物理地址
http://luodw.cc/2016/02/17/address/
Linux内存管理
http://gityuan.com/2015/10/30/kernel-memory/
/proc/iomem和/proc/ioports
http://blog.csdn.net/ysbj123/article/details/51088644
port地址空间和memory地址空间是两个分别编址的空间，都是从0地址开始
port地址也可以映射到memory空间中来，前提是硬件必须支持MMIO
iomem—I/O映射方式的I/O端口和内存映射方式的I/O端口
http://www.cnblogs.com/b2tang/archive/2009/07/07/1518175.html


协程
https://github.com/Tencent/libco


#if FOO < BAR
#error "This section will only work on UNIX systems"
#endif
http://hbprotoss.github.io/posts/cyu-yan-hong-de-te-shu-yong-fa-he-ji-ge-keng.html
https://linuxtoy.org/archives/pass.html
https://en.m.wikipedia.org/wiki/Padding_(cryptography)





 sed -e 's/collectd/\1/' *
sed只取匹配部分
http://mosquito.blog.51cto.com/2973374/1072249
http://blog.sina.com.cn/s/blog_470ab86f010115kv.html

通过sed只显示匹配行或者某些行
----- 显示1,10行
$ sed -n '1,10p' filename
----- 显示第10行
$ sed -n '10p' filename
----- 显示某些匹配行
$ sed -n '/This/p' filename
sed -n '/\<collectd\>/p' *
sed -i 's/\<Collectd/Uagent/g' *




Proxy
简单的实现，通常用于嵌入式系统
https://github.com/kklis/proxy
Golang实现的代理，包括了0复制技术等 
https://github.com/funny/proxy
dnscrypt-proxy DNS客户端与服务端的加密传输，使用libevent库
https://github.com/jedisct1/dnscrypt-proxy
ProxyChains Sock以及HTTP代理，通常用在类似TOR上面
https://github.com/haad/proxychains
https://github.com/rofl0r/proxychains-ng
基于libevent的简单代理服务
https://github.com/wangchuan3533/proxy
支持多种协议的代理服务，包括FTP、DNS、TCP、UDP等等
https://github.com/z3APA3A/3proxy
一个加密的代理服务
https://github.com/proxytunnel/proxytunnel
Vanish缓存中使用的代理服务
https://github.com/varnish/hitch
360修改的MySQL代理服务
https://github.com/Qihoo360/Atlas
Twitter提供的Memchached代理服务
https://github.com/twitter/twemproxy

UDP可靠传输方案
http://blog.codingnow.com/2016/03/reliable_udp.html
http://blog.csdn.net/kennyrose/article/details/7557917
BitCoin中使用的可靠UDP传输方案
https://github.com/maidsafe-archive/MaidSafe-RUDP
UDP方案的优缺点
https://blog.wilddog.com/?p=668




export http_proxy="http://<username>:<password>@proxy.foobar.com:8080"
export https_proxy="http://<username>:<password>@proxy.foobar.com:8080"
export ftp_proxy="http://<username>:<password>@proxy.foobar.com:8080"
export no_proxy="xxxx,xxxx"


https://github.com/wglass/collectd-haproxy  Python
https://github.com/Fotolia/collectd-mod-haproxy  C
https://github.com/funzoneq/collectd-haproxy-nbproc
https://github.com/signalfx/collectd-haproxy  ***
https://github.com/mleinart/collectd-haproxy  *

很多collectd插件的组合，很多不错的监控指标梳理
https://github.com/signalfx/integrations
https://github.com/DataDog/the-monitor
https://www.librato.com/docs/kb/collect/integrations/haproxy/
https://www.datadoghq.com/blog/monitoring-haproxy-performance-metrics/
https://github.com/mleinart/collectd-haproxy/blob/master/haproxy.py
Request: (只对HTTP代理有效)
   request_rate(req_rate)      px->fe_req_per_sec           proxy_inc_fe_req_ctr()  请求速率
   req_rate_max 请求限制速率
   req_tot 目前为止总的请求数
Response: (只对HTTP代理有效)
  'hrsp_1xx': ('response_1xx', 'derive'),
  'hrsp_2xx': ('response_2xx', 'derive'),
  'hrsp_3xx': ('response_3xx', 'derive'),
  'hrsp_4xx': ('response_4xx', 'derive'),
  'hrsp_5xx': ('response_5xx', 'derive'),
  'hrsp_other': ('response_other', 'derive'),

>>>>>backend<<<<<
Time:
  qtime (v1.5+) 过去1024个请求在队里中的平均等待时间
  rtime (v1.5+) 过去1024个请求在队里中的平均响应时间









http://savannah.nongnu.org/projects/nss-mysql
https://github.com/NigelCunningham/pam-MySQL
http://lanxianting.blog.51cto.com/7394580/1767113
https://stackoverflow.com/questions/7271939/warning-ignoring-return-value-of-scanf-declared-with-attribute-warn-unused-r
https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html
https://stackoverflow.com/questions/30813452/how-to-ignore-all-warnings-in-c
http://www.jianshu.com/p/7e84a33b46e9
https://github.com/flike/kingshard/blob/master/doc/KingDoc/kingshard_admin_api.md   微博
__attribute__((warn_unused_result))
/post/mysql-introduce.html
通过 RPM 包安装时，默认是保存到 `/var/lib/mysql` 目录下。
rpmbuild -bb SPECS/librdkafka.spec --define "__version 0.9.4" --define "__release 1"





←
-->

{% highlight text %}
{% endhighlight %}

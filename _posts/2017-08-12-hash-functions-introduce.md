---
title: Hash 函数简介
layout: post
comments: true
language: chinese
category: [program,misc]
keywords: 
description:
---


<!-- more -->


<!--
DJBHash()
 - hash += (hash << 5) + (*str++);
 + hash = ((hash << 5) + hash)  + (*str++);

https://www.byvoid.com/zhs/blog/string-hash-compare
http://www.partow.net/programming/hashfunctions/index.html


https://github.com/davidar/c-hashtable
https://github.com/larsendt/hashtable
-->

## 参考

[Murmur3 C](https://github.com/PeterScott/murmur3) C 语言实现的 Murmur3 哈希算法。

{% highlight text %}
{% endhighlight %}

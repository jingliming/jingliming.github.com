---
title: 通用规范
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---


## 命名规范

这里的命名规范包括了 `Tags` 。

可用字符为 ASCII ，包括了 `A-Z` `a-z` `0-9` `_` `@` `#` 。


## 统一错误码

错误码通过 `32bits` 指定，其中高 `16bits` 用来表示分类。

### Server(Aspire)

也就是 BootAgent 的服务端，高位是 `0x10` 。

{% highlight text %}
0x1001   AgentSN冲突无法更新
{% endhighlight %}


{% highlight text %}
{% endhighlight %}

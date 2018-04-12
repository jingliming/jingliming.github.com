---
title: React 语法简介
layout: post
comments: true
language: chinese
category: [misc]
keywords: react,introduce,syntax
description: 这里简单介绍一下 React 的语法。
---

这里简单介绍一下 React 的语法。

<!-- more -->

![react logo]({{ site.url }}/images/react-logo.png "react logo "){: .pull-center width="70%" }

## 简介

### import


好久没有写笔记了，得好好学习才行呀。

#### 1. 在B.js中引用A.js

也就是导入默认的变量。

{% highlight text %}
import A from './A';
{% endhighlight %}

此时 `A.js` 中必须有默认导出 `export default`，则 import 后面的命名可以随意取，因为总会解析到 `A.js` 中的 `export default`，例如：

{% highlight text %}
// A.js
export default 42

// B.js
import A from './A'
import MyA from './A'
import Something from './A'
{% endhighlight %}

#### 2. 使用花括号

也就是导入特定的变量。

{% highlight text %}
import {A} from './A';
{% endhighlight %}

其中花括号中的名字必须是 `A.js` 中通过 `export` 导出的名字，例如：

{% highlight text %}
export const A = 42
{% endhighlight %}

此时如果导入的是不同的名字，那么就会报错。

{% highlight text %}
// B.js
import { A } from './A'
import { myA } from './A'       // Doesn't work!
import { Something } from './A' // Doesn't work!
{% endhighlight %}

#### 3. 导入多个变量

一个模块的默认导出只能有一个，但是可以指定多个导出名，例如

{% highlight text %}
// B.js
import A, { myA, Something } from './A'
{% endhighlight %}

这里导入了默认的 A 以及 myA、Something 。

#### 4. 导入重命名

假设有如下的 `A.js` 文件。

{% highlight text %}
// A.js
export default 42
export const myA = 43
export const Something = 44
{% endhighlight %}

那么在导入时可以通过如下方式进行重命名。

{% highlight text %}
// B.js
import X, { myA as myX, Something as XSomething } from './A'
{% endhighlight %}



## 参考


{% highlight text %}
{% endhighlight %}

---
title: Golang反射机制应用场景
layout: post
comments: true
language: chinese
category: [golang]
keywords: golang,reflect
description: golang反射机制如何应用
---

简要介绍使用Golang反射的两种应用场景及代码示例

<!-- more -->

## 反射机制应用场景
反射常见应用场景有以下两种：
1、不知道接口调用哪个函数，根据传入参数在运行时确定调用的具体接口，这种需要对函数或方法反射。例如以下这种桥接模式：
{% highlight text %}
func bridge(funcPtr interface{}, args ...interface{})
{% endhighlight %}

第一个参数funcPtr以接口的形式传入函数指针，函数参数args以可变参数的形式传入，bridge函数中可以用反射来动态执行funcPtr函数。
2、不知道传入函数的参数类型，函数需要在运行时处理任意参数对象，这种需要对结构体对象反射。典型应用场景是ORM，gorom示例如下：
{% highlight text %}
type User struct {
  gorm.Model
  Name         string
  Age          sql.NullInt64
  Birthday     *time.Time
  Email        string  `gorm:"type:varchar(100);unique_index"`
  Role         string  `gorm:"size:255"` // set field size to 255
  MemberNumber *string `gorm:"unique;not null"` // set member number to unique and not null
  Num          int     `gorm:"AUTO_INCREMENT"` // set num to auto incrementable
  Address      string  `gorm:"index:addr"` // create index with name `addr` for address
  IgnoreMe     int     `gorm:"-"` // ignore this field
}

var users []User
db.Find(&users)
{% endhighlight %}
示例中Find函数不知道传入的参数是什么类型，但要能处理任意任意参数。如果类型合法返回正确的值，否则返回异常。




{% highlight text %}
{% endhighlight %}

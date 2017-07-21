---
title: Python 与 Curses
layout: post
comments: true
language: chinese
category: [python,program]
keywords: python,ncurse,curse
description:
---

<!-- more -->

如果你有许多的窗口对象(都需要刷新), 为了避免不必要的闪烁, 你可以先对各个需要刷新的窗口调用 noutrefresh(), 它将升级内在的数据结构使之匹配你所要的内容, 然后统一调用 doupdate() 来刷新屏幕.



通常，如果有 6 个参数 (pad 的 refresh 函数)，其参数及其含义如下。

window.refresh([pminrow, pmincol, sminrow, smincol, smaxrow, smaxcol])



pminrow, pmincol
    左上角的位置；

The 6 optional arguments can only be specified when the window is a pad created with newpad(). The additional parameters are needed to indicate what part of the pad and screen are involved. pminrow and pmincol specify the upper left-hand corner of the rectangle to be displayed in the pad.

sminrow, smincol, smaxrow, and smaxcol specify the edges of the rectangle to be displayed on the screen.
The lower right-hand corner of the rectangle to be displayed in the pad is calculated from the screen coordinates, since the rectangles must be the same size. Both rectangles must be entirely contained within their respective structures. Negative values of pminrow, pmincol, sminrow, or smincol are treated as if they were zero.



### curses.panel

如下是一个简单的示例，会绘制两个 panel ，而且第二个 panel 会自动移动。

{% highlight python %}
from time import sleep
import curses, curses.panel

def make_panel(h, l, y,x, str):
    win = curses.newwin(h, l, y, x)
    win.erase()
    win.box()
    win.addstr(2, 2, str)

    panel = curses.panel.new_panel(win)
    return win, panel

def test(stdscr):
    try:
        curses.curs_set(0)
    except:
        pass

    stdscr.box()
    stdscr.addstr(2, 2, "panels everywhere")
    win1, panel1 = make_panel(10,12, 5,5, "Panel 1")
    win2, panel2 = make_panel(10,12, 8,8, "Panel 2")
    curses.panel.update_panels(); stdscr.refresh()
    sleep(1)

    panel1.top(); curses.panel.update_panels(); stdscr.refresh()
    sleep(1)

    for i in range(20):
        panel2.move(8, 8+i)
        curses.panel.update_panels(); stdscr.refresh()
        sleep(0.1)

    sleep(1)

if __name__ == '__main__':
    curses.wrapper(test)
{% endhighlight %}





## 参考

可以查看下官方的介绍文档 [Curses Programming with Python](https://docs.python.org/2/howto/curses.html)，以及 Python 中的 curses 模块，可查看 [Terminal handling for character-cell displays](https://docs.python.org/2/library/curses.html) 。

关于 ncurses 的相关内容可以参考 [NCURSES Programming HOWTO](http://tldp.org/HOWTO/NCURSES-Programming-HOWTO/) 以及 [www.gnu.org](https://www.gnu.org/software/ncurses/) 中的介绍。

<!--

Terminal handling for character-cell displays
https://docs.python.org/2/library/curses.html

Terminal handling for character-cell displays
https://docs.python.org/dev/library/curses.html

A panel stack extension for curses
https://docs.python.org/dev/library/curses.panel.html

Simplified curses
https://pypi.python.org/pypi/cursed


Ncurses Programming Guide
hughm.cs.ukzn.ac.za/~murrellh/os/notes/ncurses.html

http://www.cnblogs.com/nzhl/p/5603600.html

libev+ncurse
https://lists.gnu.org/archive/html/bug-ncurses/2015-06/msg00046.html
ncurse贪吃蛇
http://www.cnblogs.com/eledim/p/4857557.html

http://www.cnblogs.com/starof/p/4703820.html
-->

{% highlight text %}
{% endhighlight %}

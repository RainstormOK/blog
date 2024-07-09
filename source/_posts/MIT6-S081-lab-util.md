---
title: MIT6.S081 lab util
date: 2024-07-09 10:26:50
tags: 
  - 国外高质量网课
  - MIT6.S081
  - Lab
---

sleep
---

注意头文件不是`stdio.h`、`stdlib.h`、`unistd.h`，参考`echo.c`的头文件。

pingpong
---

创建两个`pipe`，写入端分别为`child`和`parent`。

primes
---

如果没有使用`system calls`的经验，这个程序其实比较困难。

注意，每次调用`pipe`，都会两个生成`file descriptor`，而`xv6`的`fd`是有数量要求的。所以在`read`或`write`后，一定要及时`close`。关闭后若仍想使用`pipe`，只需要继续调用`pipe(p)`，然后`fork()`。

程序中需要有两个`pipe`存在，一个用于写入所有质数，一个用于写入数组的大小。在我的程序中，它们分别为`int p[2]`和`int pn[2]`。

还有，这个程序有执行顺序的要求，所以`parent`应该调用`wait`，等待`child`退出后，再执行程序。

我写的程序利用了递归`recursion`。主函数`fork`后，`child`将所有数字读入`pipe`，`parent`等待`child`退出后，调用函数`func_parent`。

函数`func_parent(int *p, int n)`概括如下：
- 使用`for`循环，读取管道`p`中的数字，将它们存在数组`arr`中，数组大小是参数`n`。注意关闭管道`p`的读写端。
- 调用`pipe(p)`和`pipe(pn)`。
- 调用`fork()`。
- `child`判断`arr`中从`arr[1]`到`arr[n - 1]`能否被`arr[0]`整除，如果不能，就写入管道`p`。同时记录写入管道`p`数字的个数`index`，将`index`写入管道`pn`。
- `parent`调用`wait`，等待`child`退出。接着读取管道`pn`，将参数`n`置为读取到的数字，然后递归调用`func_parent`。

这个程序的核心是，管道`p`像是流水线，进程从管道中读取数字，剔除不需要的数字，然后将数字送入下个管道`p`。
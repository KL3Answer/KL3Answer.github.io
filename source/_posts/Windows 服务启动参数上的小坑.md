---
title: Windows 服务启动参数上的小坑
date: 2017-12-08 22:33:40
tags: 
	- 错误记录
---
最近比较懒，不太想写东西，可能是传说中的~~悟道~~太懒了吧，所以准备写点~~对象~~东西，比如记录一个小坑什么的。

额。。其实这也不算是坑，严格意义上来说这是一个应该注意的地方。

首先看一张图：

![](https://raw.githubusercontent.com/KL3Answer/KL3Answer.github.io/hexo/source/temp/t_01.png)

这服务器上的MySQL的启动参数（windows 服务器），这里指定了启动时使用的配置文件，就是这个地方坑了我。

***
好吧，进入正题，上午在日志的看见一个异常
>### Error updating database.  Cause: java.sql.SQLException: The total number of locks exceeds the lock table size
具体的异常信息就不全放出来了，因为项目用的是随便搭的服务器（测试的项目），所以我猜想应该是innodb_buffer_size太小产生的问题。

在MySQL上查了一下，发现果然innodb_buffer_size只有8388608(8M)，那问题就容易解决了，于是我打算使用set来把buffer size 改成 512M，结果发现是read only的，貌似只能改配置文件了。

于是我兴冲冲的修改了MySQL安装目录下面的my.ini，然后重启了mysql，发现事情并没有那么简单，buffer size 还是没有变，然后我就简单的思索了以下，发现应该是windows服务的启动上有什么不对的，然后就发现了上面的那个。

关于解决方法，这个就很简单了，要么改变启动参数中配置文件的指向，要么该百年启动参数中配置文件的配置，然后重启。


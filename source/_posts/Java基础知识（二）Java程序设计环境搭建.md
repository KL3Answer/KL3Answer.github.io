---
title: Java基础知识（二） Java程序设计环境搭建
date: 2015-1-27 22:13:00
tags: Java基础知识
---
本文主要介绍windows下的Java开发环境（不包括web开发）的搭建。

### 一、下载JDK
一般来说，我们从[Oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html)上下载JDK。新版的JDk具有很多新特性，目前我使用的是jdk1.8.0_102。
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314195649885-1315347654.png)
进入上面的网址点击Java SE的图标可以看到上面的视图，点击右侧下载JDK安装包。

* 注：JDK（Java Development Kit）是编写Java程序的程序员使用软件，JDK中包含了JRE（Java Runtime Environment，其中由JVM和一些核心类库，你可以在安装有JRE的计算机的运行Java程序，但是进行程序设计至少还要有编译源代码的工具）和编译器等其他开发工具

### 二、安装JDK
在windows下JDk的安装非常简单，直接点击下载的安装包安装即可。
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200035698-1812508487.png)



但是要注意安装的目录不要包含除英文外的其他特殊符号（中文、日文、空格等），而且如果你在之前已经再该目录下安装了JDK，有可能会被清空之前安装的JDK（即使在点击安装之后取消），此外如果你已经安装了eclipse等工具的话jre的配置也会被清空，必须重新配置。

### 三、配置环境变量

安装完成之后，需要配置系统环境变量：控制面板-系统-高级系统设置-环境变量
 
* 新建 JAVA_HOME 变量并添加下JDK安装的路径（比如我的在C:\Program Files\Java\jdk1.8.0_102），方便日后修改JDK的安装路径
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200105541-1959735739.png)



* 新建 Path 变量 并添加下面两条（设置执行路径）
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200133541-2065649122.png)



* 新建 CLASSPATH 变量并添加，（指定类路径）
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200144276-2083226191.png)



* 配置完后运行cmd，输入java -version，若显示下面信息则表示安装和配置成功(版本号根据你安装的版本而定)。
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200157245-1865735659.png)



*注：用户变量是对当前登录的用户有效，如果你让所有登录用户都有效的话在系统变量中配置。

在安装完毕之后，我们来看一下JDK目录下的文件
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200215010-2104660431.png)



bin包含了编译器和一些其他的工具

db中放的是[apache derby](http://db.apache.org/derby/)的运行库和执行文件

include中是用于编译本地方法的文件

jre是Java运行环境文件

lib是一些核心类库文件

src.zip是类库的源文件


### 四、安装Java的IDE（Eclipse） 

当然，你也可以选择使用文本编辑器和命令行来开发Java程序。当你完成你的代码之后，在命令行中是用javac命令可以将你的.java编译成.class字节码文件，然后使用java命令运行
![](http://images2015.cnblogs.com/blog/1124237/201703/1124237-20170314200229135-673500189.png)



（注意你的类路径，在这里我使用了set classpath来将当前的文件夹临时设置为类文件加载的路径，若不设置，则使用你在上面的CLASSPATH类路径）

在上面的操作中，我使用'javac'调用了java的编译器，将Demo.java编译成了Demo.class,并使用'java'将字节码发送到Java虚拟机，由虚拟机执行了.class文件。

我一般使用IDE（Integrated Development Environment）来开发Java程序，Java的IDE个人认为比较不错的有[Eclipse](https://www.eclipse.org/downloads/)和[Intelli J IDEA](https://www.jetbrains.com/idea/),非web开发使用Eclipse足矣，因此下面主要介绍Eclispe的安装和配置（在以后的web开发中再介绍相关的Intelli J和Eclipse以及MyEclipse的相关配置）。

从上面的网址可以下载Eclipse（Eclipse是免费的）,下载完成之后可以直接运行eclipse.exe来进行Java程序的开发。启动Eclipse之后，建议先对Eclipse使用的JDK进行配置（工具栏window-preference-搜索Installed JREs-点击右边的add添加之前安装的JDK，不配置则Eclipse会使用自带的JDK），你也可以对Eclipse进行一些其他的配置（比如文件编码，字体颜色，自动提示等）

### 五、Eclipse的一些快捷键
CTRL
--------------
Ctrl+1 快速修复

Ctrl+D: 删除当前行

Ctrl+Q 定位到最后编辑的地方

Ctrl+L 定位在某行

Ctrl+O 快速显示 OutLine

Ctrl+T 快速显示当前类的继承结构

Ctrl+W 关闭当前Editer

Ctrl+K 快速定位到下一个

Ctrl+E 快速显示当前Editer的下拉列表

Ctrl+J 正向增量查找(按下Ctrl+J后,你所输入的每个字母编辑器都提供快速匹配定位到某个单词,如果没有,则在stutes line中显示没有找到了,)

Ctrl+Z 返回到修改前的状态

Ctrl+Y 与上面的操作相反

Ctrl+/ 注释当前行,再按则取消注释

Ctrl+D删除当前行。

Ctrl+Q跳到最后一次的编辑处

Ctrl+M切换窗口的大小

Ctrl+I格式化激活的元素Format Active Elements。

Ctrl+F6切换到下一个Editor

Ctrl+F7切换到下一个Perspective

Ctrl+F8切换到下一个View

CTRL+SHIFT
------
Ctrl+Shift+E 显示管理当前打开的所有的View的管理器(可以选择关闭,激活等操作)

Ctrl+Shift+/ 自动注释代码

Ctrl+Shift+\自动取消已经注释的代码

Ctrl+Shift+O 自动引导类包

Ctrl+Shift+J 反向增量查找(和上条相同,只不过是从后往前查)

Ctrl+Shift+F4 关闭所有打开的Editer

Ctrl+Shift+X 把当前选中的文本全部变为大写

Ctrl+Shift+Y 把当前选中的文本全部变为小写

Ctrl+Shift+F 格式化当前代码

Ctrl+Shift+M(先把光标放在需导入包的类名上) 作用是加Import语句

Ctrl+Shift+P 定位到对于的匹配符(譬如{}) (从前面定位后面时,光标要在匹配符里面,后面到前面,则反之)

Ctrl+Shift+F格式化文件Format Document。

Ctrl+Shift+O作用是缺少的Import语句被加入，多余的Import语句被删除。

Ctrl+Shift+S保存所有未保存的文件。

Ctrl+Shift+/ 在代码窗口中是这种/*~*/注释，在JSP文件窗口中是 <!--~-->。

Shift+Ctrl+Enter 在当前行插入空行(原理同上条)

ALT
-----
Alt+/ 代码助手完成一些代码的插入 ，自动显示提示信息

Alt+↓ 当前行和下面一行交互位置(特别实用,可以省去先剪切,再粘贴了)

Alt+↑ 当前行和上面一行交互位置(同上)

Alt+← 前一个编辑的页面

Alt+→ 下一个编辑的页面(当然是针对上面那条来说了)

Alt+Enter 显示当前选择资源(工程,or 文件 or文件)的属性

MyEclipse 快捷键4(ALT+CTRL)

Alt+CTRL+↓ 复制当前行到下一行(复制增加)

Alt+CTRL+↑ 复制当前行到上一行(复制增加)

ALT+SHIFT
-------
Alt+Shift+R 重命名

Alt+Shift+M 抽取方法

Alt+Shift+C 修改函数结构(比较实用,有N个函数调用了这个方法,修改一次搞定)

Alt+Shift+L 抽取本地变量

Alt+Shift+F 把Class中的local变量变为field变量

Alt+Shift+I 合并变量

Alt+Shift+V 移动函数和变量

Alt+Shift+Z 重构的后悔药(Undo) Shift+Enter 在当前行的下一行插入空行(这时鼠标可以在当前行的任一位置,不一定是最后)

Alt+Shift+O(或点击工具栏中的Toggle Mark Occurrences按钮) 当点击某个标记时可使本页面中其他地方的此标记黄色凸显，并且窗口的右边框会出现白色的方块，点击此方块会跳到此标记处。


fn
---
F2当鼠标放在一个标记处出现Tooltip时候按

F2则把鼠标移开时Tooltip还会显示即Show Tooltip Description。

F3跳到声明或定义的地方。

F5单步调试进入函数内部。

F6单步调试不进入函数内部，如果装了金山词霸2006则要把“取词开关”的快捷键改成其他的。

F7由函数内部返回到调用处。

F8一直执行到下一个断点。





</br>
</br>
</br>
</br>
</br>
参考书目：
</br>*Core.Java.Volume.I.Fundamentals.10th.Edition.2016.1*
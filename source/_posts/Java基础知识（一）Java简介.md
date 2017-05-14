---
title: Java基础知识（一） Java简介
date: 2015-1-13 22:13:00
tags: 
	- Java基础知识
---
Java是一门面向对象的高级编程语言，具有简单性、面向对象、分布式、健壮性、安全性、平台独立与可移植性、多线程、动态性等特点。Java与C++相关，C++源自C，而Java的大量特性就是从这两种语言继承的：Java从C继承了它的语法，从C++继承了许多面向对象的特性，当然Java也有许多自己的特点（更多介绍可以参考java核心技术卷1和java8编程官方参考）。

### 一、高级语言OR低级语言

* 高级（high-level）语言的典型特征是将多个机器语言（machine language、machine code 或称为 native code，是计算机直接识别的二进制文件，cpu通过读取这些二进制文件来执行相应的指令，具有平台相关性）封装到单行代码中，比如Java、C、C++、Pascal等，而这些语言的编译器的任务就是接收复杂的语句或表达式并生成相应的机器语言来执行这个任务。

* 低级（low-level）语言则是以机器语言中的操作与程序代码中的语句之间有非常紧密的关系为特征的，例如汇编语言（assembly language,注：JVM中的汇编语言是jasmin）。

### 二、解释性OR编译型

* 编译型语言(compiled language)可由其编译器直接编译成机器语言并直接执行，具有运行效率高和平台相关性的特点。

* 解释型语言(interpreted language)是有专门的解释器对源代码（source code）逐行解释成特定平台的机器语言并执行的语言，一般不会进行整体的编译。每一次执行解释型语言的程序都要将源代码解释成机器语言并执行，因此其运行速度较编译型语言慢，但是具有良好的可移植性和动态性。

* Java在早期原则上是解释型的，早期的Java源代码（.java文件）由其编译器一次性编译成JVM的机器语言（.class文件），并由JVM来执行这些字节码（JVM最终会将其转换成本平台支持的机器语言并由本地计算机执行），因此早期的Java程序速度相当慢。而现在的Java早已引入了JIT（just in time）技术， 此时的JVM会将一些频繁运行的代码块认定成热点代码（hot spot code），并在运行时直接将这些代码编译成本地平台的机器代码，省去了JVM再次的编译时间。综上所述，Java兼具解释型和编译型的特点（“以解释为基础，对热点进行编译”）。
* 注：更准确地来说，编程语言并没有解释型和编译型之分，C语言也可以由交互式的解释器执行，Python也可以通过一个直接的编译器直接得到机器语言，这只是编程语言的两种不同执行策略。

### 三、动态类型OR静态类型

* 动态类型语言（dynamically typed language）是指在运行期间才去做数据类型检查的语言。在用动态语言编程时，不用给变量指定数据类型，该语言会在你第一次赋值给变量时，在内部将数据类型记录下来。Python和Ruby就是一种典型的动态类型语言；

* 静态类型语言（Statically typed language）与动态类型语言刚好相反，它的数据类型检查发生在在编译阶段，也就是说在写程序时要声明变量的数据类型。C/C++、C#、Java都是静态类型语言的典型代表。

### 四、强类型OR弱类型

语言有无类型、弱类型、强类型之分。类型强弱主要指类型检查的严格程度。无类型语言（typeless language）不检查，也不区分数据和指令。

* 弱类型语言（Implicit typed language）只严格区分数据和指令，强类型语言（Explicit typed language）严格在编译器对数据类型和指令进行检查区分,在没有强制类型转换之前，不允许两种类型的变量互相操作。

* 强类型的语言是类型安全的语言。
强类型语言某种程度上速度慢于弱类型语言（没有数据类型检查），弱类型语言可以节省代码量，而强类型则更加规范，利于构建复杂项目。
常见的强类型语言有C#、Java、Python3，常见的弱类型语言有C、C++、javascript、php


* 注：语言是否动态类型与是否类型安全没有联系！
例如：Python是动态语言，是强类型定义语言（类型安全的语言）;
VBScript是动态语言，是弱类型定义语言（类型不安全的语言）;
JAVA是静态语言，是强类型定义语言（类型安全的语言）。

</br>
</br>
### 一些概念：
**Trapped error**: An execution error that immediately results in a fault.
</br>**Untrapped error**: An execution error that does not immediately result in a fault.
</br>**Forbidden error**: The occurrence of one of a predetermined
class of execution errors;Typically the improper application of an
operation to a value, such as not(3).
</br>**Well behaved**: A program fragment that will not produce
forbidden errors at run time.
</br>**Strongly checked language**: A language where no forbidden errors can
occur at run time (depending on the definition of forbidden error).
</br>**Weakly checked language**: A language that is statically checked
but provides no clear guarantee of absence of execution errors.
</br>**Statically checked language**: A language where good behavior is determined before execution.
</br>**Dynamically checked language**: A language where good behavior is enforced during execution.
</br>**Type safety**:  The property stating that programs do not cause untrapped errors.
</br>**Explicitly typed language**: A typed language where types are part of the syntax.
</br>**Implicitly typed language**: A typed language where types are not part of the syntax.








</br>
</br>
</br>
</br>
</br>
参考书目：
</br>*Core.Java.Volume.I.Fundamentals.10th.Edition.2016.1*
</br>*Princiles of Computer Organnization and Assembly Language:using the Java Virtual Machine*
</br>*Java:The Complete Reference,Ninth Edition*
</br>*Type Systems*
---
title: 关于Maven的目录结构
date: 2014-08-12 10:20:55
tags: 
	- 错误记录
---
### 先把你用的工具基本原理弄清楚再开发！！！

1. 之前别人问我一个问题，为什么他的项目在把其他项目的代码迁移过来的时候总是会有一些问题，我看了一下，他使用的工具类（Freemarker,用于生成模板）是从别的项目里直接那过来的，然后在他这边就总是运行不正常，而且没有报错信息。
看了一下，其中发现的了几个问题：
* 运行的异常其实是FileNotFoundException，是Freemarker在获取模板的时候产生的；
* 之所以没有异常信息是因为他使用的工具类在捕获异常之后没有抛出；(所以说不要完全相信别人写的工具类，鬼知道里面有多少坑。。)

2. 那发现了问题之后解决就更简单了，看了一下目录结构就知道了，他在maven项目的src/main/java下面的静态资源找不到，废话当然找不到，maven是按照一定的约定生成target的(所谓的依托于standard convention)，而你最后运行的代码是在target里，maven默认java文件夹下面都是源码(Application/library sources)，打包的时候不打包其下面的静态资源的,因为资源默认是在src/main/resources下面的(Application/Library resources)。

3. 那有人会问了，为什么他迁移的那个源项目可以这么跑呢，因为他喵的源项目不是maven项目，我也不知道这个年头除了个人开发为什么会有人用这种项目构建方式(个人开发也不会选这种方式吧。。为什么要手动导入jar)。

4. 关于Maven的目录结构，Maven官网上有关于[Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)的解释。

5. 如果不想把资源放在src/main/resources下面怎么办。自己在src/main下面建一个文件夹,参照：
>If there are other contributing sources to the artifact build, they would be under other subdirectories: for example src/main/antlr would contain Antlr grammar definition files.
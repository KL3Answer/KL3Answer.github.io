---
title: 关于Maven的一些命令
date: 2015-06-04 21:01:55
tags: 
	- 错误记录
---

 因为在之前的Maven项目中把 install 命令错用成install:install之后，浪费了小半天的时间，所以现在痛定思痛，写一些关于Maven命令的一些错误记录。

### 一
 首先是[Maven 3.5.0 的 ref](http://maven.apache.org/components/ref/3.5.0/) 的官网，基本上你想了解的关于Maven使用的细节都在上面。
### 二
 下面是正题，一般我们在构建多module项目时，项目与项目之间都是以jar包依赖的方式来管理整个项目的,那自然被依赖的项目就要事先就存在于(mvn install)Maven库中，然后部署的时候打包(mvn package)主模块就可以发布了，但是，因为我使用的是intellij，而且也已经很久没有使用了maven的命令了，所以在intellij 的图形化界面上被自己坑了，intellij 右侧的maven tab 里面点开模块之后有一大堆子选项（Lifecycle、Plugins等等），重点就是Lifecyle和Plugins里面都有install选项（当然，这两个不一样，Plugins里面的当然是 maven-install-plugin 的 goal，是用来将指定的artifact安装到repo，而Lifecycle的 install是对目标项目生成artifact并安装到repo），所以在这里就是吃了对maven生疏了的亏。。。。（不不不，我觉得是intellij的界面误导了我）

 简而言之，就是intellij 中的 Plugins 下面的install 执行的是 mvn install:install 命令，而我们需要执行的是 mvn install 命令，所以当然会出错了。

 另外，如果你和我一样执行错了命令，错误大概也是这样的：
 >[ERROR] Failed to execute goal org.apache.maven.plugins:maven-install-plugin:2.5.2:install (default-cli) on project o2o_support: The packaging for this project did not assign a file to the build artifact -> [Help 1]

### 三
总结一下，操作图形化界面的时候还是得小心，因为你根本不知道你到底有没有点错，是不是执行的你想执行的命令。



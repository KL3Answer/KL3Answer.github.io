---
title: 使用 JWT 和 session 进行权限校验
date: 2017-10-21 16:33:40
tags: 
	- 错误记录
---
好久没遇到问题了，一遇到就和一堆大问题，这几天被项目里面的遗留代码坑的焦头烂额的（比如hibernate auto-ddl改变表结构和数据的坑和大量数据导致trasaction 获取锁超时的坑，还有mysql connector 版本问题的坑），趁这个周末的有一点空闲时间，在这里记录一下关于使用JWT和Session进行用户权限校验的方式。

### 一、为什么要使用JWT
使用JWT(JSON Web Tokens)来进行权限验证的基本原理就是将验证信息以 JSON 的方式存储、编码并添加数字签名，放在 requst 的 Header 中，然后server在收到request之后就可以通过JWT中的信息来进行校验了，更多关于JWT的知识可以在JWT官网的[intruduction](https://jwt.io/introduction/)浏览。

之所这个项目中使用JWT主要是因为这个项目是前后端分离的，stateless 的 JWT 可以更方便地解决CORS问题，而且使用session的话，随着用户的增加，server端就需要更多的内存空间来存储session了，至于JWT的安全性方面的问题，这个到不是最大的影响因素，因为JWT比session在安全性方面的优势在于可以通过expiration来作废掉之前发布的JWT，让client重新申请JWT，但是同样的效果似乎也可以通过设置session的过期时间来达到吧。

### 二、JWT的短板
既然JWT的好处那么多，那它有没有什么短板呢？。

一个很尴尬的地方就是没办法直接废除掉已经发布的 JWT，因为 JWT 存储在 client 那边，而 server 这边似乎没有什么办法让 client 端直接删掉自己的 JWT，所以只能在 server 端通过其他手段来废除掉已发布的JWT（例如在数据库存储用户关联的 JWT 的 audience 来针对性的废除掉用户的 JWT）。这种情况下，使用 session 的话可能会方便一点。

### 三、为什么还使用 session
session 作为一个域对象，可以方便地存取对象，配合 Interceptor 或者 Filter 使用可以让代码更加简洁。当然，使用request或者直接使用ThreadLocal也可以，这里只是为了兼容 session 验证的模式。

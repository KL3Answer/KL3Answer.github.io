---
title: SkyWalking 初探
date: 2019-02-26 23:20:12
tags: 
	- 其他
---

![](https://i.loli.net/2019/02/26/5c74deb472b11.png)

上面这张图是我们今天的主角 SkyWalking 对它自己的描述：一个开源的可以对从服务到云设施等一系列软件进行搜集、分析、聚合、可视化的监控平台，具有上图所示的那一堆巴拉巴拉特性。

在正式介绍 SkyWalking 之前，让我们先回想一下，我们都是如何对线上服务进行问题定位和链路追踪的。如果是在代码调试阶段，方法自然多种多样，debug 或者控制台打印或者使用一些计数都可以满足。但是对于在线上运行的程序来说就不一样了：如果简单的或者是非分布式的单体服务，我们可以使用日志系统和单体监控系统。但是目前的互联网服务中，很大一部分应用都是分布式集群的方式来实现的，甚至有可能是使用不同的编程语言来实现的，在这种情况下，传统的监控方式就显得力不从心了，因为它难以观察记录一个完整的执行链路，性能分析与监控也就无从谈起了。因此，一个能够在大规模分布式系统中进行追踪的系统就显得尤为重要。

有需求自然就市场，于是这几年之间诞生了各种各样的分布式追踪系统，而这些分布式追踪系统，广义上都可以划为 APM。

这里的 APM，全称是 Application performance management，是用来监控管理软件应用的性能和可用性的软件。根据 Gartner Research 在2018年三月的 ["Magic Quadrant for Application Performance
Monitoring Suites"](http://market.tingyun.com/report/2018GartnerAPM%E9%AD%94%E5%8A%9B%E8%B1%A1%E9%99%90%E6%8A%A5%E5%91%8A.pdf)对 APM 做的定义，可以将 APM 划分为三种：
1. Digital experience monitoring (DEM)
2. Application discovery, tracing and diagnostics (ADTD)
3. Artificial intelligence for IT operations(AIOps) for applications

很明显，按照上面的划分， SkyWalking 等一众分布式追踪系统都是偏 ADTD 的。说起 ADTD， 2010 年， Google 发表了一篇名为 ["Dapper - a Large-Scale Distributed Systems Tracing Infrastructure"](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/papers/dapper-2010-1.pdf)的技术报告，其中介绍了 Google 旗下名为 Dapper 分布式追踪系统。这篇文章中的许多概念被后来的各种 APM 接受和发展，所以目前很多的分布式追踪系统在架构上与 Dapper 非常类似，总体的设计也及其相似（关于 Dapper 的具体内容我们之后再说）。
截止到 2019年2月，市面上已经存在了大量的各式各样的分布式追踪系统，其中国内有阿里的 EagleEye（不开源），京东的 Callgraph（不开源）、美团的MTrace（不开源）、新浪的Watchman（不开源）、上面提到的SkyWalking（开源），国外的则有 Twitter 的 Zipkin（开源）、naver 的 Pinpoint（开源）、Uber 的 jaeger（开源），除了分布式追踪系统之外，也诞生了 Opentracing 这个语言无关、厂商中立的标准化 API，来帮助开发者更方便地在实现了这一规范的分布式追踪系统的之间迁移与融合（Opentracing API 使用了门面模式来是帮助开发者在现有代码中集成特定的 agent ，隔离不同 APM 的 agent 之间的代码，从而减轻在不同 APM 之间迁移的工作量，当然这不是一个强制性的标准，上述的那些 APM 并不是都兼容 Opentracing API，这一部分就不展开讲了）。

对上面这几个 APM 的简单对比如图所示：

| 名称 | 厂商 | 是否开源 | 是否支持OpenTracing标准 | 代码侵入性 | 传输协议  | 可视化展示 | 性能损耗 | 扩展性 |trace查询|告警支持|存储|
|---|---|---|---|---|---|---|---|--|--|--|--|
|EagleEye|淘宝|不开源|-|-|-|-|-|-|-|-|-|
|CallGraph|京东|不开源|-|-|-|-|-|-|-|-|-|
|MTrace|美团|不开源|-|-|-|-|-|-|-|-|-|
|Watchman|新浪|不开源|-|-|-|-|-|-|-|-|-|
|SkyWalking|华为 吴晟|开源|支持|极低（agent注入）|gRPC|中|低|中|支持|支持|ES，H2,mysql,TIDB,sharding sphere|
|Pinpoint|naver|开源|不支持|极低（agent注入）|thrift|高|高|低|不支持|支持|hbase|
|Zipkin|twitter|开源|部分支持|中（请求拦截）|http，MQ|中|中|高|支持|不支持|ES，mysql,Cassandra,内存|
|Jaeger|uber|是|支持|中（请求拦截）|udp，http|中|中|高|支持|不支持|ES，kafka,Cassandra,内存|

既然有这么多可选项，那么为什么今天我们谈论是SkyWalking呢？首先，从实际业务场景出发，如果我们需要让这套追踪系统与线上运行的模块更好的契合，或者当它运行出现问题时及时地修复，我们难免需要对它进行二次开发，所以先排除掉不开源的那些，因此EagleEye 、Callgraph、MTrace、Watchman 这几个被淘汰。其次虽然在上面那个表格中各个项目各个指标之间互有胜负，但是从优先级和权重上来看，我们一般会更关心探针的性能和对现有代码的入侵性，而报表展示、传输协议等虽然也很重要，但是二次开发的难度和时间成本上相对优化探针的性能和探针收集方式修改更低一些。因此，SkyWalking 就在一众选手之中脱颖而出。

说了这么多，终于到我们今天的主角 SkyWalking 了。

SkyWalking 是由华为的开发者吴晟开源的一款 APM，目前已经进入 Apache 软件基金会。
SkyWalking 的设计目标是：
* Keep Observability（保持监控）

    * 不管在什么环境中部署，都可以提供监控

* Topology, Metric and Trace Together（拓扑结构、性能指标、追踪共存）

    主要是从可视化与用户友好的角度，这三者共存可以帮组用户更好的理解和分析目标系统

* Light Weight（轻量化）

    1. 探针性能影响最下化
    2. SkyWalking 后端最简化

* Pluggable（插件化）

    * 方便用户在不同场景下使用

* Portability（便携，方便在各种环境集成）

    * 可与各种目标系统集成

* Interop（与社区合作）

    * 借助社区的力量完善自身

SkyWalking 的架构如下所示（本文中的 SkyWalking 版本为6.X）
![](http://skywalking.apache.org/assets/frame.jpeg)

其中的主要角色有那么几个：
* 探针（Probe）。将探针插入用户的程序中，用于收集和格式化数据。
* 平台后端（Observability Analysis Platform，OAP），可集群。用于聚合、分析数据流。OAP可以以插件化的形式兼容其他格式的数据（比如 Zipkin 的格式）、其他类型的存储和集群管理。
* 存储（Storage）。用于存储 Probe 收集的数据，目前已经可以通过插件接入 Elasticsearch、H2、MySQL，也可以自行开发接入其他存储源。
* 展示（UI）。将 SkyWalking 的追踪数据可视化。

在探针方面，SkyWalking 允许用户通过多种方式使用探针，如通过语言原生的 agent 功能（比如 javaagent，该方式在有 VM 的编程语言中比较常见）自动插入探针、通过 Service mesh（该词经常和 Micro Service 一起被提起,可以简单对将 Service Mesh 理解为管理服务与服务之间通信、流控、熔断降级等功能的通讯中间层）搜集Control Panel、Data Panel 等的数据、通过第三方增强库（比如 Zipkin 的 agent）。

在 OAP 方面，它类似于 Dapper 的 collector ，不过功能相较于 collector 更为丰富，除了收集 Probe 的数据并将其存储到 Storage 之外，还承担分析和 SkyWalking 自己的 DSL 查询功能。

介绍完这些，我们来看一下 SkyWalking 的集成。
SkyWalking 的集成非常简单，拿 Java 的服务为例子，需要被追踪的服务只需要将 SkyWalking 的 Probe 的jar 包通过 javaagent 的方式加入（用于动态插入追踪代码），同时添加 SkyWalking 的 apm-toolkit 相关 jar 包（提供用于追踪的注解和其他 API），同时在需要被追踪的方法上添加 @Trace 注解或者通过 OpenTracing 的 API 添加 Trace 和 Span 生成代码（相关的 API 非常简单，这里就不展开了），如果需要跨线程追踪的话可以使用 SkyWalking 的跨线程追踪 API（对 Runnable 和 Callable 等 Java 的 异步 API 进行封装）

* OpenTracing API 方式
![](https://i.loli.net/2019/02/25/5c738d19348ac.png)
* @Trace 注解方式和 SkyWalking 自己的 API 方式
![](https://i.loli.net/2019/02/25/5c738dcd49f19.png)
* 跨线程追踪方式
![](https://i.loli.net/2019/02/25/5c738e888fddf.png)

集成完之后，再搭好 OAP 和 Storage，并完成相关配置，最基本的 SkyWalking 服务就完成了。

下面是我在推送线的代码中集成 SkyWalking 之后，在 OAP 上查看的效果：

![](https://i.loli.net/2019/02/25/5c73d6436948e.png)
![](https://i.loli.net/2019/02/25/5c73d7ce14e73.png)

第一张图是 SkyWalking 根据 Probe 收集的数据生成的跟踪树（Trace tree）的列表，跟踪树的具体结构见第二张图。可以看出 SendMessageAction 的一个实例执行了 process 方法，而 process 方法内又调用了 Jedis 的 hgetAll。
前面我们谈到了跟踪树，其实跟踪树这个概念是被我们上文中提到的 Dapper 发扬光大的，在 Dapper 之后诞生的分布式追踪系统，包括 OpenTracing 这个 标准都与 Dapper 非常相似，不管是从概念、架构还是实现方式上。所以，我们先看一下 Dapper 中的核心概念：跟踪树。
![](https://i.loli.net/2019/02/25/5c73e0332c2d2.png)

上面这张图来自之前提到的那篇 google Dapper 的论文，该图展示的是一个请求从发起，到穿过层层系统,最后返回给请求者的过程，字母标示的框框代表的是一个分布式服务中的不同的进程，此图中发起一个请求时，首先会到达前端，然后通过两个RPC到服务器B和C。B会马上返回响应，但是C需要和后端的D和E交互之后再返还给A，由A来响应最初的请求。
在上面这种场景下，我们可以通过在服务的每一个收发请求的动作上收集请求标识（message identifiers）和时间戳（timestamped events）来实现.
为了将所有记录条目与一个给定的发起者（例如，图1中的RequestX）关联上并记录所有信息，有两种经典的解决方案，第一种是黑盒(black-box)，第二种是基于标注(annotation-based)的监控方案。黑盒方案假定需要跟踪的除了上述信息之外没有额外的信息，这样使用统计回归技术来推断两者之间的关系。基于标注的方案依赖于应用程序或中间件明确地标记一个全局ID，从而连接每一条记录和发起者的请求。虽然黑盒方案比标注方案更轻便，但是他们需要更多的数据，以获得足够的精度，因为他们依赖于统计推论。基于标注的方案最主要的缺点是，需要代码植入。（顺便说一下，SkyWalking 就是使用代码植入的方式实现的分布式追踪）

前一张图中的场景可以拥跟踪树来表示成这样:
![](https://i.loli.net/2019/02/25/5c73e6f28781c.png)

在 Dapper 跟踪树结构中，树节点是整个架构的基本单元，称为 span，span 之间的连线表示一个 span 和它的父 span 之间的关联。不管一个 span 处于树中的哪个位置，它同时也是一个带有时间戳信息的包含了span 的开始结束时间，RPC的时序数据，和零个或多个特定于应用的注解的简单日志。

插入的 agent 代码会记录 span 名称，以及每个 span 的 ID 和父 span ID，以重建在一次追踪过程中不同 span 之间的关系。如果一个 span 没有父 ID 则被称为 root span。所有span都挂在一个特定的 trace 上，共用一个 trace id。

在我们简单介绍完跟踪树之后再来看 SkyWalking，一个完美的 SkyWalking 链路展示图应该是下面这样的：

![](https://i.loli.net/2019/02/25/5c73ebf30987c.png)(该图来自 SkyWalking 官网)

我们可以看到，我之前截的那张图和这一张比起来，可以说是非常单薄了。

为什么会这样呢，并不是因为我偷懒，实际上在截上面那张图的时候，我已经运行了好几个互相关联的模块并且都集成了 SkyWalking 的标准化API，但是效果就和那张图一样，不是非常理想，其原因在于：对于一些通用组件来说，SkyWalking 可以较好的搜集到跟踪信息（比如 redis 、MySQL，HTTP 请求等），但是由于我们推送线的相关模块之间的调用都是我们自己研发的组件，所以如果需要达成较好的效果，需要通过自己开发插件的方式来实现。此外，SkyWalking 在方法级别的追踪上还有一些小问题，比如静态方法无法追踪，所以，如果如果希望 SkyWalking 与现有的代码良好结合，很有可能需要进行对应的插件开发甚至是二次开发的。
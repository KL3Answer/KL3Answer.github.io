---
title: Java 8新特性 CompletableFuture
date: 2015-04-20 12:11:11
tags: 
	- Java 8 新特性
---
JDK1.5中的Future给我们提供了一个可以返回结果的多线程编程的接口，而在JDK1.8中的CompletableFuture则更加强大。

### 一、关于CompletableFuture
CompletableFuture继承于Future，但是其中又增加了五十多种方法，而且其中内置的lambda表达式使的这个类的使用更加强大和轻便。
首先你可以很简单的创建CompletableFuture

	 CompletableFuture<String> cf = new CompletableFuture<>();
不过现在对象cf和Callable并没有关系，也没有线程池也不是异步的，如果现在调用cf.get()将阻塞。所以一般在调用之前需要调用complete方法传入get()方法的以及相关方法的返回值。

当你想代表Future的任务时是非常方便的，而且没有必要去计算一些执行线程的任务上。CompletableFuture.complete()只能调用一次，后续调用将被忽略。但也有一个后门叫做CompletableFuture.obtrudeValue(…)覆盖一个新Future之前的价值，请小心使用。
	
有时你想要看到信号发生故障的情况，如你所知Future对象可以处理它所包含的结果或异常。如果你想进一步传递一些异常，可以用CompletableFuture.completeExceptionally(ex) (或者用obtrudeException(ex)这样更强大的方法覆盖前面的异常)。 completeExceptionally()也能解锁所有等待的客户端，但这一次从get()抛出异常。说到get()，也有CompletableFuture.join()方法在错误处理方面有着细微的变动。但总体上，它们都是一样的。最后也有CompletableFuture.getNow(valueIfAbsent)方法没有阻塞但是如果Future还没完成将返回默认值，这使得当构建那种我们不想等太久的健壮系统时非常有用。
最后static的方法是用completedFuture(value)来返回已经完成Future的对象，当测试或者写一些适配器层时可能非常有用。
### 二、获取CompletableFuture
手动地创建CompletableFuture是我们唯一的选择吗？不一定。就像一般的Futures，我们可以关联存在的任务，同时CompletableFuture使用工厂方法：

	
	static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
	static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
	static CompletableFuture<Void> runAsync(Runnable runnable);
	static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);

无参方法Executor是以…Async结尾同时将会使用ForkJoinPool.commonPool()(全局的，在JDK8中介绍的通用池），这适用于CompletableFuture类中的大多数的方法。runAsync()易于理解，注意它需要Runnable，因此它返回CompletableFuture<Void>作为Runnable不返回任何值。如果你需要处理异步操作并返回结果，使用Supplier<>

	final CompletableFuture<String> future = CompletableFuture.supplyAsync(new Supplier<String>() {
	    @Override
	    public String get() {
	        //...long running...
	        return "42";
	    }
	}, executor);
使用lambda表达式简写：


	finalCompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
	    //...long running...
	    return "42";
	}, executor);
或者：

	final CompletableFuture<String> future =
	    CompletableFuture.supplyAsync(() -> longRunningTask(params), executor);

### 三、转换和作用于CompletableFuture(thenApply)
Scala和JavaScript都允许future完成时允许注册异步回调，直到它准备好我们才要等待和阻止它。我们可以简单地说：运行这个函数时就出现了结果。此外，我们可以叠加这些功能，把多个future组合在一起等。例如如果我们从String转为Integer，我们可以转为在不关联的前提下从CompletableFuture到 CompletableFuture<Integer。这是通过thenApply()的方法：


	CompletableFuture<U> thenApply(Function<? super T,? extends U> fn);
	CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn);
	CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor);

如前所述...Async版本提供对CompletableFuture的大多数操作，因此我将在后面的部分中跳过它们。记住，第一个方法将在future完成的相同线程中调用该方法，而剩下的两个将在不同的线程池中异步地调用它。
让我们来看看thenApply()的工作流程：

	CompletableFuture<String> f1 = //...
	CompletableFuture<Integer> f2 = f1.thenApply(Integer::parseInt);
	CompletableFuture<Double> f3 = f2.thenApply(r -> r * r * Math.PI);

或在一个声明中：

	CompletableFuture<Double> f3 =
	    f1.thenApply(Integer::parseInt).thenApply(r -> r * r * Math.PI);
这里，你会看到一个序列的转换，从String到Integer再到Double。但最重要的是，这些转换既不立即执行也不停止。这些转换既不立即执行也不停止。他们只是记得，当原始f1完成他们所执行的程序。如果某些转换非常耗时，你可以提供你自己的Executor来异步地运行他们。注意,此操作相当于Scala中的一元map。

### 四、运行完成的代码（thenAccept/thenRun）

	CompletableFuture<Void> thenAccept(Consumer<? super T> block);
	CompletableFuture<Void> thenRun(Runnable action);
在future的管道里有两种典型的“最终”阶段方法。他们在你使用future的值的时候做好准备，当 thenAccept()提供最终的值时，thenRun执行 Runnable，这甚至没有方法去计算值。例如：

	future.thenAcceptAsync(dbl -> log.debug("Result: {}", dbl), executor);
	log.debug("Continuing");

…Async变量也可用两种方法，隐式和显式执行器，我不会过多强调这个方法。
thenAccept()/thenRun()方法并没有发生阻塞（即使没有明确的executor)。它们像一个事件侦听器/处理程序，你连接到一个future时，这将执行一段时间。”Continuing”消息将立即出现，尽管future甚至没有完成。
### 五、单个CompletableFuture的错误处理
到目前为止，我们只讨论计算的结果。那么异常呢？我们可以异步地处理它们吗？当然！

	CompletableFuture<String> safe =
    future.exceptionally(ex -> "We have a problem: " + ex.getMessage());

exceptionally()接受一个函数时，将调用原始future来抛出一个异常。我们会有机会将此异常转换为和Future类型的兼容的一些值来进行恢复。safe进一步的转换将不再产生一个异常而是从提供功能的函数返回一个String值。
一个更加灵活的方法是handle()接受一个函数，它接收正确的结果或异常：

	CompletableFuture<Integer> safe = future.handle((ok, ex) -> {
	    if (ok != null) {
	        return Integer.parseInt(ok);
	    } else {
	        log.warn("Problem", ex);
	        return -1;
	    }
	});

handle()总是被调用，结果和异常都非空，这是个一站式全方位的策略。

### 六、结合（链接）这两个futures（thenCompose()）
有时你想运行一些future的值（当它准备好了），但这个函数也返回了future。CompletableFuture足够灵活地明白我们的函数结果现在应该作为顶级的future，对比CompletableFuture<CompletableFuture>。方法 thenCompose()相当于Scala的flatMap：

	CompletableFuture<U> thenCompose(Function<? super T,CompletableFuture<U>> fn);

…Async变化也是可用的，在下面的事例中，仔细观察thenApply()(map)和thenCompose()（flatMap）的类型和差异，当应用calculateRelevance()方法返回CompletableFuture：

	CompletableFuture<Document> docFuture = //...
	
	CompletableFuture<CompletableFuture<Double>> f =
	    docFuture.thenApply(this::calculateRelevance);
	
	CompletableFuture<Double> relevanceFuture =
	    docFuture.thenCompose(this::calculateRelevance);
	
	//...

	private CompletableFuture<Double> calculateRelevance(Document doc)  //...
thenCompose()允许构建健壮的和异步的管道，没有阻塞和等待的中间步骤。

### 七、两个futures的转换值(thenCombine())
当thenCompose()用于链接一个future时依赖于thenCombine，当他们都完成之后就结合两个独立的futures：

	<U,V> CompletableFuture<V> thenCombine(CompletableFuture<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)

…Async变量也是可用的，假设你有两个CompletableFuture，一个加载Customer另一个加载最近的Shop。他们彼此完全独立，但是当他们完成时，想要使用它们的值来计算Route。

	CompletableFuture<Customer> customerFuture = loadCustomerDetails(123);
	CompletableFuture<Shop> shopFuture = closestShop();
	CompletableFuture<Route> routeFuture =
    customerFuture.thenCombine(shopFuture, (cust, shop) -> findRoute(cust, shop));

	//...
	
	private Route findRoute(Customer customer, Shop shop) //...

请注意，在Java 8中可以用(cust, shop) -> findRoute(cust, shop)简单地代替this::findRoute方法的引用：

	customerFuture.thenCombine(shopFuture, this::findRoute);
你也知道，我们有customerFuture 和 shopFuture。那么routeFuture包装它们然后“等待”它们完成。当他们准备好了，它会运行我们提供的函数来结合所有的结果(findRoute())。当两个基本的futures完成并且 findRoute()也完成时，这样routeFuture将会完成。

### 八、等待所有的 CompletableFutures 完成
如果不是产生新的CompletableFuture连接这两个结果，我们只是希望当完成时得到通知，我们可以使用thenAcceptBoth()/runAfterBoth()系列的方法，（…Async 变量也是可用的）。它们的工作方式与thenAccept() 和 thenRun()类似，但是是等待两个futures而不是一个：

	CompletableFuture<Void> thenAcceptBoth(CompletableFuture<? extends U> other, BiConsumer<? super T,? super U> block)
CompletableFuture<Void> runAfterBoth(CompletableFuture<?> other, Runnable action)
想象一下上面的例子，这不是产生新的 CompletableFuture，你只是想要立刻发送一些事件或刷新GUI。这可以很容易地实现：thenAcceptBoth():

	customerFuture.thenAcceptBoth(shopFuture, (cust, shop) -> {
	    final Route route = findRoute(cust, shop);
    //refresh GUI with route
});
也许有些人会问自己一个问题：为什么我不能简单地阻塞这两个futures呢？ 就像：

	Future<Customer> customerFuture = loadCustomerDetails(123);
	Future<Shop> shopFuture = closestShop();
	findRoute(customerFuture.get(), shopFuture.get());
你当然可以这么做。但是最关键的一点是CompletableFuture是允许异步的，它是事件驱动的编程模型而不是阻塞并急切地等待着结果。所以在功能上，上面两部分代码是等价的，但后者没有必要占用一个线程来执行。
### 九、等待第一个 CompletableFuture 来完成任务
另一个有趣的事是CompletableFutureAPI可以等待第一个（与所有相反）完成的future。当你有两个相同类型任务的结果时就显得非常方便，你只要关心响应时间就行了，没有哪个任务是优先的。API方法(…Async变量也是可用的）：

	CompletableFuture<Void> acceptEither(CompletableFuture<? extends T> other, Consumer<? super T> block)
	CompletableFuture<Void> runAfterEither(CompletableFuture<?> other, Runnable action)

作为一个例子，你有两个系统可以集成。一个具有较小的平均响应时间但是拥有高的标准差，另一个一般情况下较慢，但是更加容易预测。为了两全其美（性能和可预测性）你可以在同一时间调用两个系统并等着谁先完成。通常这会是第一个系统，但是在进度变得缓慢时，第二个系统就可以在可接受的时间内完成：
复制代码 代码如下:

	CompletableFuture<String> fast = fetchFast();
	CompletableFuture<String> predictable = fetchPredictably();
	fast.acceptEither(predictable, s -> {
	    System.out.println("Result: " + s);
	});
s代表了从fetchFast()或是fetchPredictably()得到的String。我们不必知道也无需关心。

### 十、完整地转换第一个系统
applyToEither()算是 acceptEither()的前辈了。当两个futures快要完成时，后者只是简单地调用一些代码片段，applyToEither()将会返回一个新的future。当这两个最初的futures完成时，新的future也会完成。API有点类似于(…Async 变量也是可用的)：

	CompletableFuture<U> applyToEither(CompletableFuture<? extends T> other, Function<? super T,U> fn)

这个额外的fn功能在第一个future被调用时能完成。我不确定这个专业化方法的目的是什么，毕竟一个人可以简单地使用：fast.applyToEither(predictable).thenApply(fn)。

	CompletableFuture<String> fast = fetchFast();
	CompletableFuture<String> predictable = fetchPredictably();
	CompletableFuture<String> firstDone =
    fast.applyToEither(predictable, Function.<String>identity());

第一个完成的future可以通过运行。请注意，从客户的角度来看，两个futures实际上是在firstDone的后面而隐藏的。客户端只是等待着future来完成并且通过applyToEither()使得当最先的两个任务完成时通知客户端。
### 十一、多种结合的CompletableFuture
我们现在知道如何等待两个future来完成（使用thenCombine()）并第一个完成(applyToEither())。但它可以扩展到任意数量的futures吗？的确，使用static辅助方法：

	static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)
	static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)

allOf()当所有的潜在futures完成时，使用了一个futures数组并且返回一个future（等待所有的障碍）。另一方面anyOf()将会等待最快的潜在futures。
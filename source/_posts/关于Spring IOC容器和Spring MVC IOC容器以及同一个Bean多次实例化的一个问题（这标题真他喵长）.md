---
title: 关于Spring IOC容器和Spring MVC IOC容器以及同一个Bean多次实例化的一个问题（这标题真他喵长）
date: 2017-08-04 21:30:00
tags: 
	- 错误记录
---
### 一、发现的问题
好吧，这次是我接手的另外一个项目，不过这一次没有之前的那么坑了，因为这个只有一个架子在。。。
也就是基本上只有一个pom.xml和web.xml和最基本的spring的配置文件，连数据库的表都还没建，不过这也好，至少不会被坑了（我以为。。）。

哈哈哈哈，我还是被坑了，他喵的连最基本的配置文件里也有坑。。。你是竞争对手派过来的卧底吗？

嗯嗯，具体的问题就是：

之前的一直用Quartz来做定时服务，近几天我用了Spring的@Scheduled来写一个情况比较简单的定时服务(用Quartz就得写一堆xml文件，麻烦死了。。)，
结果发现定时服务的执行逻辑和我预想的逻辑不一致，更具体来说，就是我的定时服务先把一个表的数据清空，然后插入新的数据，而且是一次定时只执行一次,并且是串行化的。但是我查看数据库却发现存在重复数据，同一条数据会存在两条完全相同的记录。


###  二、分析过程
看了一下服务器的日志，没有异常信息，而且打断点调试之后发现每一次删除表的数据都是成功的，也就是说，重复的数据不是旧的数据，而是每一次定时都被执行了两次。

这样看来，应该是@Scheduled注解上面出了问题，那我们来看一下Spring对这个注解做了什么。

首先，我们来看一下@Scheduled注解：
![](http://ou5y7q62a.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20170804213800.png)
可见，@Scheduled是被ScheduledAnnotationBeanPostProcessor注册的，而且是由@EnableScheduling注解指引自动注册的，我们接着看一下@EnableScheduling注解是怎么回事：

![](http://ou5y7q62a.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20170804222429.png)
注意图中的这一句话：
> \* This enables detection of @{@link Scheduled} annotations on any Spring-managed<br/>
> \* bean in the container.  For example, given a class {@code MyTask}

也就是说你在注入Spring IOC容器的Bean上面加上@EnableScheduling，那么这个Bean里面的被@Scheduled注解的方法就会被扫描(不过上图也说了，你也可以在Configuration classses上面加这个注解，效果应该差不多？)

好的，那就是说，定时任务所在类被@EnableScheduling注解之后，并且在定时任务方法上加上@Scheduled注解，那么Spring IOC容器在初始化的时候就会调用ScheduledAnnotationBeanPostProcessor来注册定时任务了。

现在我们来看一下ScheduledAnnotationBeanPostProcessor到底干了什么
![](http://ou5y7q62a.bkt.clouddn.com/3.png)
上面这个是它的描述，可知在Spring扫描到@EnableScheduling之后会注册一个ScheduledAnnotationBeanPostProcessor,通过断点，我们可以知道这个类中的关键方法是这个：
![](http://ou5y7q62a.bkt.clouddn.com/4.png)
这个方法使用反射获取被@Scheduled注解的方法，并对这个方法如下调用：(因为太长了截不了图)


    protected void processScheduled(Scheduled scheduled, Method method, Object bean) {
		try {
			Assert.isTrue(void.class.equals(method.getReturnType()),
					"Only void-returning methods may be annotated with @Scheduled");
			Assert.isTrue(method.getParameterTypes().length == 0,
					"Only no-arg methods may be annotated with @Scheduled");

			if (AopUtils.isJdkDynamicProxy(bean)) {
				try {
					// Found a @Scheduled method on the target class for this JDK proxy ->
					// is it also present on the proxy itself?
					method = bean.getClass().getMethod(method.getName(), method.getParameterTypes());
				}
				catch (SecurityException ex) {
					ReflectionUtils.handleReflectionException(ex);
				}
				catch (NoSuchMethodException ex) {
					throw new IllegalStateException(String.format(
							"@Scheduled method '%s' found on bean target class '%s', " +
							"but not found in any interface(s) for bean JDK proxy. Either " +
							"pull the method up to an interface or switch to subclass (CGLIB) " +
							"proxies by setting proxy-target-class/proxyTargetClass " +
							"attribute to 'true'", method.getName(), method.getDeclaringClass().getSimpleName()));
				}
			}

			Runnable runnable = new ScheduledMethodRunnable(bean, method);
			boolean processedSchedule = false;
			String errorMessage = "Exactly one of the 'cron', 'fixedDelay(String)', or 'fixedRate(String)' attributes is required";

			// Determine initial delay
			long initialDelay = scheduled.initialDelay();
			String initialDelayString = scheduled.initialDelayString();
			if (StringUtils.hasText(initialDelayString)) {
				Assert.isTrue(initialDelay < 0, "Specify 'initialDelay' or 'initialDelayString', not both");
				if (this.embeddedValueResolver != null) {
					initialDelayString = this.embeddedValueResolver.resolveStringValue(initialDelayString);
				}
				try {
					initialDelay = Integer.parseInt(initialDelayString);
				}
				catch (NumberFormatException ex) {
					throw new IllegalArgumentException(
							"Invalid initialDelayString value \"" + initialDelayString + "\" - cannot parse into integer");
				}
			}

			// Check cron expression
			String cron = scheduled.cron();
			if (StringUtils.hasText(cron)) {
				Assert.isTrue(initialDelay == -1, "'initialDelay' not supported for cron triggers");
				processedSchedule = true;
				String zone = scheduled.zone();
				if (this.embeddedValueResolver != null) {
					cron = this.embeddedValueResolver.resolveStringValue(cron);
					zone = this.embeddedValueResolver.resolveStringValue(zone);
				}
				TimeZone timeZone;
				if (StringUtils.hasText(zone)) {
					timeZone = StringUtils.parseTimeZoneString(zone);
				}
				else {
					timeZone = TimeZone.getDefault();
				}
				this.registrar.addCronTask(new CronTask(runnable, new CronTrigger(cron, timeZone)));
			}

			// At this point we don't need to differentiate between initial delay set or not anymore
			if (initialDelay < 0) {
				initialDelay = 0;
			}

			// Check fixed delay
			long fixedDelay = scheduled.fixedDelay();
			if (fixedDelay >= 0) {
				Assert.isTrue(!processedSchedule, errorMessage);
				processedSchedule = true;
				this.registrar.addFixedDelayTask(new IntervalTask(runnable, fixedDelay, initialDelay));
			}
			String fixedDelayString = scheduled.fixedDelayString();
			if (StringUtils.hasText(fixedDelayString)) {
				Assert.isTrue(!processedSchedule, errorMessage);
				processedSchedule = true;
				if (this.embeddedValueResolver != null) {
					fixedDelayString = this.embeddedValueResolver.resolveStringValue(fixedDelayString);
				}
				try {
					fixedDelay = Integer.parseInt(fixedDelayString);
				}
				catch (NumberFormatException ex) {
					throw new IllegalArgumentException(
							"Invalid fixedDelayString value \"" + fixedDelayString + "\" - cannot parse into integer");
				}
				this.registrar.addFixedDelayTask(new IntervalTask(runnable, fixedDelay, initialDelay));
			}

			// Check fixed rate
			long fixedRate = scheduled.fixedRate();
			if (fixedRate >= 0) {
				Assert.isTrue(!processedSchedule, errorMessage);
				processedSchedule = true;
				this.registrar.addFixedRateTask(new IntervalTask(runnable, fixedRate, initialDelay));
			}
			String fixedRateString = scheduled.fixedRateString();
			if (StringUtils.hasText(fixedRateString)) {
				Assert.isTrue(!processedSchedule, errorMessage);
				processedSchedule = true;
				if (this.embeddedValueResolver != null) {
					fixedRateString = this.embeddedValueResolver.resolveStringValue(fixedRateString);
				}
				try {
					fixedRate = Integer.parseInt(fixedRateString);
				}
				catch (NumberFormatException ex) {
					throw new IllegalArgumentException(
							"Invalid fixedRateString value \"" + fixedRateString + "\" - cannot parse into integer");
				}
				this.registrar.addFixedRateTask(new IntervalTask(runnable, fixedRate, initialDelay));
			}

			// Check whether we had any attribute set
			Assert.isTrue(processedSchedule, errorMessage);
		}
		catch (IllegalArgumentException ex) {
			throw new IllegalStateException(
					"Encountered invalid @Scheduled method '" + method.getName() + "': " + ex.getMessage());
		}
	}
这个方法有点儿长，但是思路很清晰:
![](http://ou5y7q62a.bkt.clouddn.com/5.png)
从上面可以看出，定时任务最后被加入ScheduledTaskRegistrar中了，来看一下这个类：
![](http://ou5y7q62a.bkt.clouddn.com/6.png)
这个类只是用来存放定时任务的（使用List来存储的）,其中的核心代码如下：
![](http://ou5y7q62a.bkt.clouddn.com/7.png)
在ScheduledTaskRegistrar被实例化之后，调用afterPropertiesSet(),afterPropertiesSet()调用TaskScheduler的schedule方法并将返回结果放入set中。
看一下schedule方法：
![](http://ou5y7q62a.bkt.clouddn.com/8.png)
到这里，我们大概了解了整个过程，所以说定时任务的执行最后是由一个继承了Delayed和Future的类定时执行的（ScheduledFuture），而且ScheduledFuture在容器中也是只会注册一次,一次定时也只会执行一次。
看来定时任务被执行两次的锅不是这个定时工具的问题（明显是我的思考方向出了问题，这个被广泛使用的东西应该不会出现这么严重的问题）。

于是,我去Spring的官网看了一下文档，其中有这么一段话引起了我的注意：
![](http://ou5y7q62a.bkt.clouddn.com/9.png)
我的代码中没有使用@Configurable，但是我猜测可能是我使用定时注解的类被实例化两次造成的结果。
再看看代码，我使用annotation-driven和component-scan来注入Bean，那么来看一下IOC的具体过程。

首先来看一下Spring初始化的过程：
    
 一个Spring  MVC项目自然是通过DispactherServlet作为入口的，先看一下它的静态代码块：
![](http://ou5y7q62a.bkt.clouddn.com/10.png)
这个没什么神奇的，就是加载配置文件而已，然后我们看一下DispatcherServlet的初始化代码：
![](http://ou5y7q62a.bkt.clouddn.com/11.png)
![](http://ou5y7q62a.bkt.clouddn.com/12.png)
使用了父类的构造函数，用来加载applicationContext,没什么好看的。
那我们看一看这个Servlet的init()方法：
好像没有。。那我们来看一下它父类FrameworkServlet的init():
也没有，那看一看FrameworkServlet的父类HttpServletBean的init()方法：
![](http://ou5y7q62a.bkt.clouddn.com/13.png)
终于找到了，看注释可以知道，是由HttpServletBean的子类覆盖initServletBean()来做初始化工作的，
找一找，这个覆盖是由FrameworkServlet完成的,贴图太累了，上代码：


    @Override
    protected final void initServletBean() throws ServletException {
        getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
        }
        long startTime = System.currentTimeMillis();

        try {
            this.webApplicationContext = initWebApplicationContext();
            initFrameworkServlet();
        }
        catch (ServletException ex) {
            this.logger.error("Context initialization failed", ex);
            throw ex;
        }
        catch (RuntimeException ex) {
            this.logger.error("Context initialization failed", ex);
            throw ex;
        }

        if (this.logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
                    elapsedTime + " ms");
        }
    }
applicationContext是由initWebApplicationContext()方法完成加载：

    protected WebApplicationContext initWebApplicationContext() {
		WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
		WebApplicationContext wac = null;

		if (this.webApplicationContext != null) {
			// A context instance was injected at construction time -> use it
			wac = this.webApplicationContext;
			if (wac instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
				if (!cwac.isActive()) {
					// The context has not yet been refreshed -> provide services such as
					// setting the parent context, setting the application context id, etc
					if (cwac.getParent() == null) {
						// The context instance was injected without an explicit parent -> set
						// the root application context (if any; may be null) as the parent
						cwac.setParent(rootContext);
					}
					configureAndRefreshWebApplicationContext(cwac);
				}
			}
		}
		if (wac == null) {
			// No context instance was injected at construction time -> see if one
			// has been registered in the servlet context. If one exists, it is assumed
			// that the parent context (if any) has already been set and that the
			// user has performed any initialization such as setting the context id
			wac = findWebApplicationContext();
		}
		if (wac == null) {
			// No context instance is defined for this servlet -> create a local one
			wac = createWebApplicationContext(rootContext);
		}

		if (!this.refreshEventReceived) {
			// Either the context is not a ConfigurableApplicationContext with refresh
			// support or the context injected at construction time had already been
			// refreshed -> trigger initial onRefresh manually here.
			onRefresh(wac);
		}

		if (this.publishContext) {
			// Publish the context as a servlet context attribute.
			String attrName = getServletContextAttributeName();
			getServletContext().setAttribute(attrName, wac);
			if (this.logger.isDebugEnabled()) {
				this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
						"' as ServletContext attribute with name [" + attrName + "]");
			}
		}

		return wac;
	}
通过断点调试，发现DispatcherServlet走的是无参的构造，所以wac为null：
![](http://ou5y7q62a.bkt.clouddn.com/14.png)
进入findWebApplicationContext():
![](http://ou5y7q62a.bkt.clouddn.com/15.png)
findWebApplicationContext()返回null：
![](http://ou5y7q62a.bkt.clouddn.com/16.png)
进入createWebApplicationContext():
![](http://ou5y7q62a.bkt.clouddn.com/17.png)
关键一步configureAndRefreshWebApplicationContext：
![](http://ou5y7q62a.bkt.clouddn.com/18.png)
通过注释可以知道，其实initPropertySources()会在refresh()之后被调用，在这里调用一次只是为postProcessWebApplicationContext()做准备，其中refresh()实际上用于加载初始化bean(隐藏的好深。。)，进入refresh():
![](http://ou5y7q62a.bkt.clouddn.com/19.png)
如果你进入prepareRefresh()就可以看到里面做了一次initPropertySources()动作来载入配置资源，但是我们现在关心的是bean的初始化，接着进入obtainFreshBeanFactory,其代码如下：

    /**
	 * Tell the subclass to refresh the internal bean factory.
	 * @return the fresh BeanFactory instance
	 * @see #refreshBeanFactory()
	 * @see #getBeanFactory()
	 */
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}
接着进入refreshBeanFactory()：
![](http://ou5y7q62a.bkt.clouddn.com/20.png)
可以知道是在loadBeanDefinitions()中读取的配置文件，进去看看：
![](http://ou5y7q62a.bkt.clouddn.com/21.png)
。。。。一层套一层，看来读取xml文件似乎是在initBeanDefinitionReader()和loadBeanDefinitions()里面完成的，进入initBeanDefinitionReader()一看什么都没有，那么loadBeanDefinitions()里面呢：
![](http://ou5y7q62a.bkt.clouddn.com/22.png)
离读取文件越来越近了，看看reader.loadBeanDefinitions()都干了什么：
里面调用了内部的方法，最后是酱紫的：
![](http://ou5y7q62a.bkt.clouddn.com/23.png)
尼玛。。又一个loadBeanDefinitions。。
反正最后读取xml文件的代码是下面这个：

    public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
		}

		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<EncodedResource>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
这样子的：

    protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
			Document doc = doLoadDocument(inputSource, resource);
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
以及这样子的：

    protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
				getValidationModeForResource(resource), isNamespaceAware());
	}
还有这样子的：

    @Override
	public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
			ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

		DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
		if (logger.isDebugEnabled()) {
			logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
		}
		DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
		return builder.parse(inputSource);
	}
还有更多。。。。
反正最后就是通过流读取xml文件返回Document对象就是了（mdzz，就不应该放这些的源码。。。）,然后registerBeanDefinitions(),其中解析xml的关键一步在下面：

    protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						parseDefaultElement(ele, delegate);
					}
					else {
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
 delegate.parseCustomElement()用于解析spring自己的xml标签，关键的一个方法：

    @Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		return findParserForElement(element, parserContext).parse(element, parserContext);
	}   
为对应的使用对象的parser,因为注入bean使用了component-scan，我们进入ComponentScanBeanDefinitionParser的parse()方法：

    @Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		String[] basePackages = StringUtils.tokenizeToStringArray(element.getAttribute(BASE_PACKAGE_ATTRIBUTE),
				ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

		// Actually scan for bean definitions and register them.
		ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
		Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
		registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

		return null;
	}

很容易知道scanner.doScan()这里执行了扫描并注入的动作:

    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
			for (BeanDefinition candidate : candidates) {
				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
				candidate.setScope(scopeMetadata.getScopeName());
				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
				if (candidate instanceof AbstractBeanDefinition) {
					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
				}
				if (candidate instanceof AnnotatedBeanDefinition) {
					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
				}
				if (checkCandidate(beanName, candidate)) {
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
					definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					beanDefinitions.add(definitionHolder);
					registerBeanDefinition(definitionHolder, this.registry);
				}
			}
		}
		return beanDefinitions;
	}

现在我们又看见了一堆完全没有注释的代码了。。。
还好，我已经帮你找出findCandidateComponents()这个方法是包扫描的主要执行者，再进入看看：

    public Set<BeanDefinition> findCandidateComponents(String basePackage) {
		Set<BeanDefinition> candidates = new LinkedHashSet<BeanDefinition>();
		try {
			String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
					resolveBasePackage(basePackage) + "/" + this.resourcePattern;
			Resource[] resources = this.resourcePatternResolver.getResources(packageSearchPath);
			boolean traceEnabled = logger.isTraceEnabled();
			boolean debugEnabled = logger.isDebugEnabled();
			for (Resource resource : resources) {
				if (traceEnabled) {
					logger.trace("Scanning " + resource);
				}
				if (resource.isReadable()) {
					try {
						MetadataReader metadataReader = this.metadataReaderFactory.getMetadataReader(resource);
						if (isCandidateComponent(metadataReader)) {
							ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
							sbd.setResource(resource);
							sbd.setSource(resource);
							if (isCandidateComponent(sbd)) {
								if (debugEnabled) {
									logger.debug("Identified candidate component class: " + resource);
								}
								candidates.add(sbd);
							}
							else {
								if (debugEnabled) {
									logger.debug("Ignored because not a concrete top-level class: " + resource);
								}
							}
						}
						else {
							if (traceEnabled) {
								logger.trace("Ignored because not matching any filter: " + resource);
							}
						}
					}
					catch (Throwable ex) {
						throw new BeanDefinitionStoreException(
								"Failed to read candidate component class: " + resource, ex);
					}
				}
				else {
					if (traceEnabled) {
						logger.trace("Ignored because not readable: " + resource);
					}
				}
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
		}
		return candidates;
	}

这里面的关键又在于this.resourcePatternResolver.getResources(packageSearchPath),再深入,发现起作用的类是PathMatchingResourcePatternResolver，看看它的getResources()方法：

    @Override
	public Resource[] getResources(String locationPattern) throws IOException {
		Assert.notNull(locationPattern, "Location pattern must not be null");
		if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
			// a class path resource (multiple resources for same name possible)
			if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
				// a class path resource pattern
				return findPathMatchingResources(locationPattern);
			}
			else {
				// all class path resources with the given name
				return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
			}
		}
		else {
			// Only look for a pattern after a prefix here
			// (to not get fooled by a pattern symbol in a strange prefix).
			int prefixEnd = locationPattern.indexOf(":") + 1;
			if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
				// a file pattern
				return findPathMatchingResources(locationPattern);
			}
			else {
				// a single resource with the given name
				return new Resource[] {getResourceLoader().getResource(locationPattern)};
			}
		}
	}
这里我们又发现了findAllClassPathResources和findPathMatchingResources好像才是干活的那个：（这里只选了findAllClassPathResources，反正基本是一致的）

    protected Resource[] findAllClassPathResources(String location) throws IOException {
		String path = location;
		if (path.startsWith("/")) {
			path = path.substring(1);
		}
		Enumeration<URL> resourceUrls = getClassLoader().getResources(path);
		Set<Resource> result = new LinkedHashSet<Resource>(16);
		while (resourceUrls.hasMoreElements()) {
			URL url = resourceUrls.nextElement();
			result.add(convertClassLoaderURL(url));
		}
		return result.toArray(new Resource[result.size()]);
	}

到这里，Spring IOC容器注入bean的原理基本浮出水面。（并没有。。好像只写了其中一部分吧）

### 三、捋一下思路
写到这里，我们整理一下思路：<br/>
（1）发现了一个问题：@Scheduled注解的方法被重复执行；
（2）@Scheduled注解及其后面的执行代码都没问题，问题应该在于bean被实例化两次；
（3）通过源码，我们发现bean实例化在xml文件中的入口是component-scan；
（4）我们又跟踪了一遍源码，源码没有问题；
（5）由（4）推出问题应该是在配置文件上面（因为我的代码不可能会导致这个问题）；
（6）看了配置文件，果然发现了问题：里面出现了两次component-scan(分别在spring mvc 和spring的配置文件里)，而且扫描的包名是同一个；
（7）回忆一下我们之前看的源码，无论是registerComponents()还是doScan()还是其之后加载类的方法，都没有做去重处理；
（8）也就是说，源头在于这个傻逼的配置文件（以前就有的，不关我事。。）扫描了两次包，从而导致的bean实例化两次，进而导致定时任务重复执行。

### 四、解决方式
好吧，我又被坑了？？？<br/>
解决倒是很简单，删掉其中一个context:component-scan，只留下一个扫描全部即。

嗯，就这样。不过好像有点不对啊，一个是Spring MVC的配置文件，另一个是Spring的配置文件，删掉可以吗，试一下看看，果然不行，因为我记得Spring MVC的IOC容器和Spring IOC容器并不是同一个（而且这个傻逼的web.xml文件同时配置了ContextLoaderListener和DispactherServlet。。。）,从上面的源码中可以知道可以存在多个ApplicationContext，而且Spring IOC容器应该会早于Spring MVC的（ContextLoaderListener先于DispactherServlet创建），两者是父子关系。

而且根据官方文档的说法，使用Spring MVC和Spring的时候只需要在web.xml中配置DispatcherServlet即可，
于是，我删掉了ContextLoaderListener的配置，发现果然定时任务恢复正常了。

或者，老一点版本的Spring好像必须配置ContextLoaderListener,此时的话，可以使用如下的方式配置：

    <context:component-scan base-package="xxx.xxx.xxx">  
       <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>  
    </context:component-scan>  

    其中expression里面的就是你要排除的注解。
	

### 五、最后还要说一下
其实一开始的时候，我就发现问题出在配置文件上面，只不过我对Spring的源码有一点好奇，就顺便跟踪源码走了一下，所以我在这里想说的是：Spring被这个项目的原作者用成这样真是（╯‵□′）╯︵┴─┴ 
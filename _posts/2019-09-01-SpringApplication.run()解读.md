### SpringApplication.run()干了啥

#### run()干了啥

#####创建`SpringApplication`对象

```java
public static void main(String[] args) {
		SpringApplication.run(StdeplearnApplication.class, args);
	}
```

#####利用创建好的`SpringApplication`对象,调用`run`方法

```java
public static ConfigurableApplicationContext run(Class<?> primarySource,
			String... args) {
		return run(new Class<?>[] { primarySource }, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources,
                                                 String[] args) {
  return new SpringApplication(primarySources).run(args);
}
```

#####`SpringApplication` 创建对象。

######版本基础1.X

```java
//构造方法
public SpringApplication(Object... sources) {
		initialize(sources);
}
```

```java
private void initialize(Object[] sources) {
  	//1.保存住配置类
		if (sources != null && sources.length > 0) {
			this.sources.addAll(Arrays.asList(sources));
		}
  	//2.判断当前是否是WEB应用
		this.webEnvironment = deduceWebEnvironment();
  	//3.从类路径下找到META-IN/spring.factories配置的所有ApplicationContextInitializer,然后保存
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
  	//4.从类路径下找到META-INF/spring.factories配置的所有ApplicationListener,然后保存起来
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
  	//5.从多个配置类中找到有main方法的主配置类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

######版本基础2.X

```java
public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
}
```

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = deduceWebApplicationType();
  	//下面代码同1.X 代码完全相同
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

#####`run 方法`

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		FailureAnalyzers analyzers = null;
		configureHeadlessProperty();
  	//1.从类路径下META-INF/spring.factories获取SpringApplicationRunListeners
		SpringApplicationRunListeners listeners = getRunListeners(args);
    //2.遍历回调SpringApplicationRunListeners的starting()事件
		listeners.starting();
		try {
      //3.封装命令行参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
      //4.准备环境，完成后遍历回调SpringApplicationRunListeners中的environmentPrepared事件
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
      //5 banner 展示，你可以在console 上看到启动图标
			Banner printedBanner = printBanner(environment);
      //6 根据是否是web 环境来决定窗前web 还是普通IOC容器 1.x 和 2.x 实现是不一样的。2.x 对REACTIVE 类型容器做了支持（源码区别见下面的一个子项） 
			context = createApplicationContext();
			analyzers = new FailureAnalyzers(context);
      // 7. 准备上下我呢环境，将environment 保存到Ioc容器中
      // 7.1 执行applyInitializers();遍历回调之前保存的ApplicationContextInitializer 的initialize() 事件
      // 7.2 遍历回调SpringApplicationRunListeners 中的contextPrepared()事件
      // 7.3 prepareContext 完成后遍历回调SpringApplicationRunListeners 中的contextLoaded()事件
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
      // 8.刷新容器，进行组建的扫描，创建，加载等
			refreshContext(context);
      // 9 从Ioc容器中火去所有的ApplicationRunner 和 CommandLineRunner 进行回调（先回调完ApplicationRunner,在回调CommandLineRunner）
			afterRefresh(context, applicationArguments);
      // 10 遍历回调 SpringApplicationRunListeners 中的finished()事件
			listeners.finished(context, null);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
      // 11.启动完成，返回Ioc 容器
			return context;
		}
		catch (Throwable ex) {
			handleRunFailure(context, listeners, analyzers, ex);
			throw new IllegalStateException(ex);
		}
	}
```

###### 1.x 的IOC容器类别

```java
//1.x 的 createApplicationContext
protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				contextClass = Class.forName(this.webEnvironment
						? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, "
								+ "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiate(contextClass);
	}
```

###### 2.X IOC 容器类别,支持了REACTIVE容器

```java
protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
        // servlet 3.0 传统的
				case SERVLET:
					contextClass = Class.forName(DEFAULT_WEB_CONTEXT_CLASS);
					break;
        // reactive 响应式服务（新的）
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
          //默认jar  应用容器
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, "
								+ "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
```

######  prepareContext 解读

```java

```

###### refreshContext 解读

```java
refresh(context);
if (this.registerShutdownHook) {
    try {
        context.registerShutdownHook();
    }
    catch (AccessControlException ex) {
        // Not allowed in some environments.
    }
}

refresh 熟悉不熟不熟悉。spring里面最核心的是啥。不就是它么
protected void refresh(ApplicationContext applicationContext) {
	Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
	((AbstractApplicationContext) applicationContext).refresh();
} 
AbstractApplicationContext.refresh(); //spring容器初始化核心方法,在这里之后就是spring-context里面的东西了
```




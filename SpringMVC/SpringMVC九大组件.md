# SpringMVC的9大组件概述

```java
   //DispatcherServlet中的定义的9个接口，指定规范，提供了非常强的扩展性

    /** 文件上传解析器. */
	@Nullable
	private MultipartResolver multipartResolver;

	/** 区域信息解析器. */
	@Nullable
	private LocaleResolver localeResolver;

	/** 主题解析器 */
	@Nullable
	private ThemeResolver themeResolver;

	/** hanlderMapping */
	@Nullable
	private List<HandlerMapping> handlerMappings;

	/** HandlerAdapter */
	@Nullable
	private List<HandlerAdapter> handlerAdapters;

	/** 异常解析器 */
	@Nullable
	private List<HandlerExceptionResolver> handlerExceptionResolvers;

	/** 请求到视图名的解析器 */
	@Nullable
	private RequestToViewNameTranslator viewNameTranslator;

	/** 允许重定向携带数据的 */
	@Nullable
	private FlashMapManager flashMapManager;

	/** 视图解析器  */
	@Nullable
	private List<ViewResolver> viewResolvers;
```

```java
    在onRefresh方法中有initStrategies方法，在这里面呢进行初始化。
        1.为什么不把initStrategies中的东西放在onRefresh中，避免过渡耦合和方便扩展。要是在onRefresh中添加新的刷新内容，可以保证隔离，别的地方调用initStrategies方法，可以很方便

    protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}

	/*
	组件的初始化：去容器中找这个组件，没找到就用默认的配置
	 */
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
```


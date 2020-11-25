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

## 1.HandlerMapping

![](resource\HandlerMapping1.png)

## 2.HandlerAdapter

![](resource\HandlerAdapter.png)

### HandlerAdapter接口

```
public interface HandlerAdapter {
    //是否支持传入的handler
	boolean supports(Object handler);

    //处理请求
	@Nullable
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

    //资源最后的修改值
	long getLastModified(HttpServletRequest request, Object handler);

}
```

```java
//HttpRequestHandlerAdapter、SimpleServletHandlerAdapter、SimpleControllerHandlerAdapter都是调用Handler里面固定的方法
public class HttpRequestHandlerAdapter implements HandlerAdapter {

	@Override
	public boolean supports(Object handler) {
		return (handler instanceof HttpRequestHandler);
	}

	@Override
	@Nullable
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		((HttpRequestHandler) handler).handleRequest(request, response);
		return null;
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		if (handler instanceof LastModified) {
			return ((LastModified) handler).getLastModified(request);
		}
		return -1L;
	}

}

```

### RequestMappingHandlerAdapter

```java
@Override
	public void afterPropertiesSet() {
		// Do this first, it may add ResponseBody advice beans
		initControllerAdviceCache();

		if (this.argumentResolvers == null) {
			List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
            //用于给处理器方法和注释了@ModelAttribute的方法设置参数
			this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
		}
		if (this.initBinderArgumentResolvers == null) {
			List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();
			//用于给@initBinder的方法设置参数
            this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
		}
		if (this.returnValueHandlers == null) {
			List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
            //将处理器的返回值变成ModeAndView对象
			this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
		}
	}

```

```java
	private void initControllerAdviceCache() {
		if (getApplicationContext() == null) {
			return;
		}

        //获取所有的@ControllerAdviceBean的bean
		List<ControllerAdviceBean> adviceBeans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());

		List<Object> requestResponseBodyAdviceBeans = new ArrayList<>();

		for (ControllerAdviceBean adviceBean : adviceBeans) {
			Class<?> beanType = adviceBean.getBeanType();
			if (beanType == null) {
				throw new IllegalStateException("Unresolvable type for ControllerAdviceBean: " + adviceBean);
			}
            //找到所有注释了@ModelAttribute方法没有标注RequestMapping的方法
			Set<Method> attrMethods = MethodIntrospector.selectMethods(beanType, MODEL_ATTRIBUTE_METHODS);
			if (!attrMethods.isEmpty()) {
				this.modelAttributeAdviceCache.put(adviceBean, attrMethods);
			}
            //查找注释了@InitBinder的方法
			Set<Method> binderMethods = MethodIntrospector.selectMethods(beanType, INIT_BINDER_METHODS);
			if (!binderMethods.isEmpty()) {
				this.initBinderAdviceCache.put(adviceBean, binderMethods);
			}
            //查找蛛丝了@ResponseBodyAdvice接口的类
			if (RequestBodyAdvice.class.isAssignableFrom(beanType) || ResponseBodyAdvice.class.isAssignableFrom(beanType)) {
				requestResponseBodyAdviceBeans.add(adviceBean);
			}
		}

		if (!requestResponseBodyAdviceBeans.isEmpty()) {
			this.requestResponseBodyAdvice.addAll(0, requestResponseBodyAdviceBeans);
		}

		if (logger.isDebugEnabled()) {
			int modelSize = this.modelAttributeAdviceCache.size();
			int binderSize = this.initBinderAdviceCache.size();
			int reqCount = getBodyAdviceCount(RequestBodyAdvice.class);
			int resCount = getBodyAdviceCount(ResponseBodyAdvice.class);
			if (modelSize == 0 && binderSize == 0 && reqCount == 0 && resCount == 0) {
				logger.debug("ControllerAdvice beans: none");
			}
			else {
				logger.debug("ControllerAdvice beans: " + modelSize + " @ModelAttribute, " + binderSize +
						" @InitBinder, " + reqCount + " RequestBodyAdvice, " + resCount + " ResponseBodyAdvice");
			}
		}
	}
```

```java
@Override
	protected ModelAndView handleInternal(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ModelAndView mav;
		checkRequest(request);

		// Execute invokeHandlerMethod in synchronized block if required.
		if (this.synchronizeOnSession) {
			HttpSession session = request.getSession(false);
			if (session != null) {
				Object mutex = WebUtils.getSessionMutex(session);
				synchronized (mutex) {
					mav = invokeHandlerMethod(request, response, handlerMethod);
				}
			}
			else {
				// No HttpSession available -> no mutex necessary
				mav = invokeHandlerMethod(request, response, handlerMethod);
			}
		}
		else {
			// No synchronization on session demanded at all...
			mav = invokeHandlerMethod(request, response, handlerMethod);
		}

		if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
			if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
				applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
			}
			else {
				prepareResponse(response);
			}
		}

		return mav;
	}
```


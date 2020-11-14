# 1.DispatcherServlet架构

- DispatcherServlet继承结构图

  <img src="resource\servlet继承结构图.png" style="zoom:60%;" />

```java
	FrameworkServlet#doGet方法重写父类HttpServlet中的doGet方法
	@Override
	protected final void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}
	
	protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
			doService(request, response);
	}

    //doService方法没有实现，留给子类实现
    protected abstract void doService(HttpServletRequest request, HttpServletResponse response)
			throws Exception;

    //DispatcherServlet#doService方法
    @Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

			doDispatch(request, response);
	
	}
```

## 1.1 servlet

```java
public interface Servlet {

    //传入ServletConfig类型的参数，容器传入进去的，web.xml中定义Servlet通过init-param标签配置的参数
    public void init(ServletConfig config) throws ServletException;
    
    //用于获取ServletCOnfig
    public ServletConfig getServletConfig();

    //处理请求
    public void service(ServletRequest req, ServletResponse res)throws ServletException, IOException;

    //获得servlet相关的信息
    public String getServletInfo();

    //销毁
    public void destroy();
}

```

## 1.2 GenericServlet 

是Servlet的默认实现

- 实现了ServletConfig接口

- 提供了无参的init方法

  ```java
   public void init(ServletConfig config) throws ServletException {
  		this.config = config;
  		this.init();
      }
    
      public void init() throws ServletException {
  
      }
  ```

- 提供了log方法

## 1.3 HttpServlet

```java
//主要就是将ServletRequest转换为HttpServletRequest,ServletResponse转化为HttpServletResponse，然后调用service方法，类似适配器 
public void service(ServletRequest req, ServletResponse res)
	throws ServletException, IOException
    {
	HttpServletRequest	request;
	HttpServletResponse	response;
	
	try {
	    request = (HttpServletRequest) req;
	    response = (HttpServletResponse) res;
	} catch (ClassCastException e) {
	    throw new ServletException("non-HTTP request or response");
	}
	service(request, response);
    }


    protected void service(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException
    {
	String method = req.getMethod();

	if (method.equals(METHOD_GET)) {
	    long lastModified = getLastModified(req);
	    if (lastModified == -1) {
		// servlet doesn't support if-modified-since, no reason
		// to go through further expensive logic
		doGet(req, resp);
	    } else {
		long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
		if (ifModifiedSince < (lastModified / 1000 * 1000)) {
		    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
		    maybeSetLastModified(resp, lastModified);
		    doGet(req, resp);
		} else {
		    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
		}
	    }

	} else if (method.equals(METHOD_HEAD)) {
	    long lastModified = getLastModified(req);
	    maybeSetLastModified(resp, lastModified);
	    doHead(req, resp);

	} else if (method.equals(METHOD_POST)) {
	    doPost(req, resp);
	    
	} else if (method.equals(METHOD_PUT)) {
	    doPut(req, resp);	
	    
	} else if (method.equals(METHOD_DELETE)) {
	    doDelete(req, resp);
	    
	} else if (method.equals(METHOD_OPTIONS)) {
	    doOptions(req,resp);
	    
	} else if (method.equals(METHOD_TRACE)) {
	    doTrace(req,resp);
	    
	} else {
	    //
	    // Note that this means NO servlet supports whatever
	    // method was requested, anywhere on this server.
	    //

	    String errMsg = lStrings.getString("http.method_not_implemented");
	    Object[] errArgs = new Object[1];
	    errArgs[0] = method;
	    errMsg = MessageFormat.format(errMsg, errArgs);
	    
	    resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
	}
    }
    
```

## 1.4 HttpServletBean

**没有涉及处理请求，只是负责创建，看处理请求可跳过**

各个Servlet的作用：

- HttpServletBean

　　主要做一些初始化的工作，将web.xml中配置的参数设置到Servlet中。比如servlet标签的子标签init-param标签中配置的参数。

- FrameworkServlet

  将Servlet与Spring容器上下文关联。其实也就是初始化FrameworkServlet的属性webApplicationContext，这个属性代表   SpringMVC上下文，它有个父类上下文，既web.xml中配置的ContextLoaderListener监听器初始化的容器上下文。

- DispatcherServlet

　　初始化各个功能的实现类。比如异常处理、视图处理、请求映射处理等。

<img src="resource\HttpServletBean.png" style="zoom:80%;" />

## 1.5 FrameWorkServlet

又将doGet，dopost等方法全部都放在processRequest方法中执行

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;

		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

		initContextHolders(request, localeContext, requestAttributes);

		try {
           //模板方法留给子类执行
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
```

# 2.doDispatch源码

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
                //1.检查是否有文件上传请求
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				//2.决定当前的请求用那个类来处理,即controller
				mappedHandler = getHandler(processedRequest);
                //3.如果没找到处理器处理，就抛出异常
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

                //4.拿到能执行这个类的所有方法的适配器（反射工具）
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// 5.适配器执行目标方法，将目标方法执行完成之后都会封装成ModeAndView
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            //6.根据方法完成返回的ModeAndView，转发到对应的页面，而且ModeAndView中的数据都在request域中
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}

```

## 2.1 getHandler()方法

<img src="resource\概念理解.png" style="zoom:60%;" />

```java
   //handlerMappings：保存了能处理那些方法的映射信息，找到执行方法的类
   protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```

## 2.2 getHandlerAdapter()方法

<img src="resource\handlermapping.png" style="zoom:60%;" />

处理器只要有标了注解的方法，就能用，在RequestMappingHandlerAdapter中匹配

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
```

## 2.3 handle()方法

```java
    public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

		return handleInternal(request, response, (HandlerMethod) handler);
	}

//调用handleInternal里面的InvokeHandlerMethod方法

    @Override
	protected ModelAndView handleInternal(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
				mav = invokeHandlerMethod(request, response, handlerMethod);
    }


    @Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
        //设置一些属性

			invocableMethod.invokeAndHandle(webRequest, mavContainer);

	}

	public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
		
	}


public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

    //获取方法参数
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		if (logger.isTraceEnabled()) {
			logger.trace("Arguments: " + Arrays.toString(args));
		}
    //进行调用
		return doInvoke(args);
	}

```

## 2.4 processDispatchResult()方法

称为视图渲染过程：将数据在页面中展示

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {
    
    //调用render进行渲染
			render(mv, request, response);
}


protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {

		View view;
		String viewName = mv.getViewName();
		if (viewName != null) {
			// 对视图进行解析，根据视图名，获得视图
			view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
			if (view == null) {
				throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
						"' in servlet with name '" + getServletName() + "'");
			}
		}
		else {
			// No need to lookup: the ModelAndView object contains the actual View object.
			view = mv.getView();
			if (view == null) {
				throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
						"View object in servlet with name '" + getServletName() + "'");
			}
		}

		// Delegate to the View object for rendering.
		if (logger.isTraceEnabled()) {
			logger.trace("Rendering view [" + view + "] ");
		}
		try {
			if (mv.getStatus() != null) {
				response.setStatus(mv.getStatus().value());
			}
			view.render(mv.getModelInternal(), request, response);
		}
		catch (Exception ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Error rendering view [" + view + "]", ex);
			}
			throw ex;
		}
	}


	@Nullable
	protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
			Locale locale, HttpServletRequest request) throws Exception {

		if (this.viewResolvers != null) {
            //遍历所有的ViewResolver，这里只有一个ViewResolver，就是在Spring文件里面配置的InternalViewResolver对象
			for (ViewResolver viewResolver : this.viewResolvers) {
				View view = viewResolver.resolveViewName(viewName, locale);
				if (view != null) {
					return view;
				}
			}
		}
		return null;
	}


public View resolveViewName(String viewName, Locale locale) throws Exception {
		//判断是否被缓存过
        if (!isCache()) {
			return createView(viewName, locale);
		}
		else {
			Object cacheKey = getCacheKey(viewName, locale);
			View view = this.viewAccessCache.get(cacheKey);
			if (view == null) {
				synchronized (this.viewCreationCache) {
					view = this.viewCreationCache.get(cacheKey);
					if (view == null) {
						// 创建视图
						view = createView(viewName, locale);
						if (view == null && this.cacheUnresolved) {
							view = UNRESOLVED_VIEW;
						}
                        //创建完成之后，放进缓存，方便下次调用
						if (view != null && this.cacheFilter.filter(view, viewName, locale)) {
							this.viewAccessCache.put(cacheKey, view);
							this.viewCreationCache.put(cacheKey, view);
						}
					}
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace(formatKey(cacheKey) + "served from cache");
				}
			}
			return (view != UNRESOLVED_VIEW ? view : null);
		}
	}


	@Override
	protected View createView(String viewName, Locale locale) throws Exception {
		// If this resolver is not supposed to handle the given view,
		// return null to pass on to the next resolver in the chain.
		if (!canHandle(viewName, locale)) {
			return null;
		}

		// 判断是否有重定向需求，redirect
		if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
			String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length());
			RedirectView view = new RedirectView(redirectUrl,
					isRedirectContextRelative(), isRedirectHttp10Compatible());
			String[] hosts = getRedirectHosts();
			if (hosts != null) {
				view.setHosts(hosts);
			}
			return applyLifecycleMethods(REDIRECT_URL_PREFIX, view);
		}

		// 判断是否有转发请求， forward
		if (viewName.startsWith(FORWARD_URL_PREFIX)) {
			String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length());
			InternalResourceView view = new InternalResourceView(forwardUrl);
			return applyLifecycleMethods(FORWARD_URL_PREFIX, view);
		}

		// 如果没有前缀，用父类对象创建
		return super.createView(viewName, locale);
	}


//render方法中的view.render(mv.getModelInternal(), request, response)
	public void render(@Nullable Map<String, ?> model, HttpServletRequest request,
			HttpServletResponse response) throws Exception {

		if (logger.isDebugEnabled()) {
			logger.debug("View " + formatViewName() +
					", model " + (model != null ? model : Collections.emptyMap()) +
					(this.staticAttributes.isEmpty() ? "" : ", static attributes " + this.staticAttributes));
		}

		Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
		prepareResponse(request, response);
        //渲染要给页面输出的所有数据
		renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
	}

@Override
	protected void renderMergedOutputModel(
			Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

		// 将模型中的数据放在请求域中
		exposeModelAsRequestAttributes(model, request);

		// Expose helpers as request attributes, if any.
		exposeHelpers(request);

		// Determine the path for the request dispatcher.
		String dispatcherPath = prepareForRendering(request, response);

		// Obtain a RequestDispatcher for the target resource (typically a JSP).
		RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
		if (rd == null) {
			throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
					"]: Check that the corresponding file exists within your web application archive!");
		}

		// If already included or response already committed, perform include, else forward.
		if (useInclude(request, response)) {
			response.setContentType(getContentType());
			if (logger.isDebugEnabled()) {
				logger.debug("Including [" + getUrl() + "]");
			}
			rd.include(request, response);
		}

		else {
			// Note: The forwarded resource is supposed to determine the content type itself.
			if (logger.isDebugEnabled()) {
				logger.debug("Forwarding to [" + getUrl() + "]");
			}
            //进行转发
			rd.forward(request, response);
		}
	}

```

视图解析器得到视图对象，视图对象进行转发和重定向页面，视图对象才能渲染视图


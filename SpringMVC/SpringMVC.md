# SpringMVC基础

## HelloWorld运行细节

### 1.运行流程：

-  客户端点击链接会发送http://localhost：8080/springmvc/hello
- 来到Tomcat服务器
- SpringMVC收到前端控制器的所有请求
- 来看请求地址和@RequestMapping标注的那个匹配，来找到到底使用那个类的那个方法
- 方法执行完成之后会有一个返回值，SpringMVC认为这个返回值就是要去的页面地址
- 拿到方法的返回值之后，用视图解析器进行拼串得到完整的页面地址。
- 拿到页面地址，前端控制器帮我们转发页面。

### 2.RequestMapping

- 告诉SpringMVC这个方法用来处理什么请求

- /可以省略，即使省略了，也是默认从当前项目下开始。习惯加上比较好

### 3.不指定spring的配置文件位置

<img src="resource\默认文件.png" style="zoom: 50%;" />

### 4./和/*的区别

- /：拦截所有请求，不拦截jsp页面，*.jsp请求

- /*：拦截所有请求，拦截jsp页面，*.jsp请求

- *.jsp是tomcat的做的事

- 服务器中的大web.xml中有一个DefaultServlet，它的是url-pattern=/,是Tomcat中处理静态资源的（除了jsp和servlet之外的都是静态资源），前端控制器中的/禁用了tomcat中的DefaultServlet，

- 前端控制器url-pattern=/，静态资源例如index.html来到dispatcherServlet看到这个方法的RequestMapping是这个index.html,所以不能访问了
- 服务器中有默认的JspServlet的配置，所以可以访问*.jsp

## RequestMapping

### method属性

<img src="resource\method方法.png" style="zoom: 80%;" />

指定请求的方式，4xx都是客户端的错误，客户端的请求方式不对，路径不对。

### params属性

```java
    必须指定请求参数里面包含username属性
    @RequestMapping(value = "/handle03",params = {"username"})
	必须指定请求参数里面不带username属性
    @RequestMapping(value = "/handle03",params = {"!username"})
```

### heders属性

和params一样，可以指定参数，可以指定浏览器访问

### consumes属性

只接受内容类型是那种的请求，规定请求头中的Content-Type

### procuces属性

告诉浏览器发挥类型是什么，给响应的头上加上Content-Type:text/html,charset=utf-8

### Ant风格的模糊匹配

- ？：代替任意一个字符

- *：任意多个字符，和一层路径

- **：代替多层路径

  模糊和精确匹配的时候，精确优先

```
"/handle0?"，可以匹配handle01,handle02,...handle09
```

## Rest

- 传统的URL地址

```
/getBook?id=1 查询图书

/deleteBook?id=1 删除图书

/updateBook?id=1 更新图书

/addBook添加图书

```

- Rest风格的URL地址，/资源名/资源标识符

```
/book/1   :GET 查询1号图书
/book/1   :PUT 更新1号图书
/book/1   :DELETE 删除1号图书
/book     :POST   添加图书
```

但是页面上只能发起GET和POST请求，其他的请求没办法使用。

```java

    @RequestMapping(value = "/book",method = RequestMethod.POST)
    public String addBook(){
        System.out.println("添加了图书");
        return "success";
    }

    @RequestMapping(value = "/book/{bid}",method = RequestMethod.DELETE)
    public String deleteBook(@PathVariable("bid") Integer id){
        System.out.println("删除了图书"+id);
        return "success";
    }

    @RequestMapping(value = "/book/{bid}",method = RequestMethod.PUT)
    public String updateBook(@PathVariable("bid") Integer id){
        System.out.println("更新了图书"+id);
        return "success";
    }

    @RequestMapping(value = "/book/{bid}",method = RequestMethod.GET)
    public String getBook(@PathVariable("bid") Integer id){
        System.out.println("查询到了"+id+"图书");
        return "success";
    }
```

```xml-dtd
在web.xml中配置过滤器，转换请求
  <filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  
  index.jsp中添加查询
  
  <a href="book/1">查询图书</a><br/>
  <form action="book" method="post">
    	<input type="submit" value="添加图书">
   </form><br/>
   <form action="book/1" method="post">
   	 	<input name="_method" value="delete">
    	<input type="submit" value="删除图书">
   </form><br/>
   <form action="book/1" method="post">
   		<input name="_method" value="put">
    	<input type="submit" value="更新图书">
   </form><br/>


出错的话，在success.jsp加上，isErrorPage="true" ，在Tomcat 8.0以上会报错
```

## SpringMVC如何获取请请求信息

- @RequestParam:获取请求参数

默认获取请求参数：直接给方法入参上写一个和请求参数名相同的变量。这个变量来接收请求参数的值。

```java
    @RequestMapping(value = "/handle05")
    public String handle05(String username){
        System.out.println("handle05"+username);
        return "success";
    }

    <a href="handle05?username=qwe">测试RequestParams</a><br/>
```

使用之后，相当于username = request.getParameter("user")

```java
 @RequestMapping(value = "/handle05")
    public String handle05(@RequestParam("user") String username){
        System.out.println("handle05"+username);
        return "success";
    }
```

@RequestHeader:获取请求头中的值

```
@RequestHeader("User-Agent") String userAgent
等于userAgent = request.getHeader("User-Agent")
```

@CookieValue

## 处理乱码

提交可能乱码：

- 请求乱码:

  - GET请求：改server.xml,在8080端口处URIEncoding="UTF-8"

  - Post请求：在第一次请求参数之前设置：request.setCharacterEncoding("UTF-8")

    ```xml
    <!--在web.xml中加入，一般放在其他filter之前，否则无法生效--> 
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
          <!-- 指定解决POST请求乱码-->
          <param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
          <!-- 解决响应乱码-->
          <param-name>forceEncoding</param-name>
          <param-value>true</param-value>
        </init-param>
      </filter>
    
      <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
      </filter-mapping>
    ```

    

- 响应乱码：response.setContentType("text/html;charset=utf-8")

## 给页面输出数据

- 可以在方法处传入Map,Model或者ModelMap,给这些参数里面保存的所有数据都会放在请求域(Request)中，可以在页面中获得

  关系：Map,Model,ModelMap：最终都是BindingAwareModelMap在工作。

- 返回值可以变成ModelAndView类型,里面有ModelMap，也有view。放在请求域中：

  ```java
  public class ModelAndView {
  
  	/** View instance or view name String. */
  	@Nullable
  	private Object view;
  
  	/** Model Map. */
  	@Nullable
  	private ModelMap model;
  
  	/** Optional HTTP status for the response. */
  	@Nullable
  	private HttpStatus status;
  
  	/** Indicates whether or not this instance has been cleared with a call to {@link #clear()}. */
  	private boolean cleared = false;
  }
  ```

- SpringMVC提供了一种可以临时给Session保存数据的方式，使用一个注解@SessionAttributes(value = "msg")，标在类上。给BindingAwareModelMap中保存的数据，同时给Session中保存一份。Value指定保存数据时要给session中放的数据的key,后来不推荐@SessionAttributes使用，可能会引发异常。给session中放数据请使用原生API。

```java
@SessionAttributes(value = "msg")
@Controller
public class OutputController {


    @RequestMapping("/output/handle01")
    public String handler01(Map<String,Object> map){
        map.put("msg","你好");
        return "success";
    }
}
```

## ModeAttribute

- SpringMVC要封装请求参数的Book对象不应该是自己new出来的。而应该是【从数据库中】拿到准备好的对象，再来使用这个对象封装请求参数

- 方法位置：这个方法会提前于目标方法先运行。可以提前查出数据库中图书的信息。
- 参数位置：可以获取对象。

## mvc:annotation-driven

开启自动注册后；

- RequestMappingHandlerMapping、RequestMappingHandlerAdapter、ExceptionHandlerExceptionResolver三个bean会自动注册

- 支持使用ConversionService实例对表单参数进行类型转换

- 支持使用@NumberFormat annotation、@DateTimeFormat注解完成数据类型格式化

- 支持使用@Valid注解对JavaBean实例进行JSR303验证

- 支持使用@RequestBody和@ResponseBody注解

<img src="resource\annotation-driver.png" style="zoom:80%;" />

- 两个注解都没配置的情况

  ![](resource\都没配的情况.png)

无法访问静态资源，只能访问动态资源

- 开启mvc:default-servlet-handler

  ![](resource\default-servlet.png)
  
- 只开启mvc:annotation-driven

  ![](resource\只有annoatation.png)



## 访问AJAX数据

```xml
  //引入坐标
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.9.0</version>
    </dependency>
```

```java
 //将数据返回
    @ResponseBody
    @RequestMapping("/getallajax")
    public  Collection<Employee> ajaxGetAll(){
        Collection<Employee> all = employeeDao.getAll();
        return all;
    }
```

![](resource\ajax运行结果.png)

```java
    @RequestMapping("/testRequestBody")
    public String testRequestBody(@RequestBody String body){
        System.out.println(body);
        return "success";
    }
```

- RequestBody：接受json数据，封装为对象

- ResponseBody：可以把对象转化为json数据，返回给浏览器

## 文件上传

导入的maven坐标

```xml
<dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3</version>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.0</version>
    </dependency>
```

```java
    @RequestMapping("/upload")
    public String upload(@RequestParam(value = "username") String username,
                         @RequestParam("headerimg") MultipartFile file,
                                 Model model)  {
        System.out.println("上传文件的信息"+file.getName());
        try {
            file.transferTo(Paths.get("D:\\360安全浏览器下载\\" + file.getOriginalFilename()));
            model.addAttribute("msg","文件上传成功");
        } catch (IOException e) {
            model.addAttribute("msg","文件上传失败");
            e.printStackTrace();
        }
        return "forward:/file.jsp";
    }
```

```xml
<body>
${msg}
<form action="${ctp}/upload" method="post" enctype="multipart/form-data">
    用户头像:<input type="file" name="headerimg"><br/>
    用户名:<input type="text" name="username"><br/>
    <input type="submit">
</form>
</body>
```



# SpringMVC数据绑定

WebDataBinder主要类

<img src="resource\databinder.png" style="zoom:80%;" />

1. SpringMVC将ServletRequest对象及目标方法的入参实例传递给WebDataBinderFactory实例，创建DataBinder对象

2. DataBinder调用装配在SpringMVC上下文的ConversionService组件进行数据类型转换、数据格式化工作。将Servlet中的请求信息填充到入参对象中

3. 调用Validator组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果BindingData对象

4. SpringMVC抽取BindingResult中的入参对象和校验错误对象，将它们赋给处理方法的响应入参

# SpringMVC数据校验

```java
   //限定日期格式，
	@DateTimeFormat(pattern = "yyyy-mm-dd")
	private Date birth;

   //假设页面，为了显示方便提交的工资是10,000.98
   @NumberFormat(pattern = "#,###,###.##")
   private Double salary;
```

- JSR303标准

# SpringMVC拦截器

- SpringMVC提供了拦截器机制，允许运行目标方法之前进行一些拦截工作，或者目标方法运行之后进行一些其他处理。

![](resource\handler.png)

- preHandle:在目标方法运行之前调用；返回boolean;return true;(chain.doFilter())放行。

- postHandle：在目标方法运行之后调用：目标方法调用之后

- afterCompletion:在整个请求完成之后；来到目标页面之后，chain.doFilter放行，资源响应后执行

拦截器是一个接口

实现HandlerIntercepter接口

preHandle->目标方法->postHandle->success.jsp->afterCompletion

![](resource\interceptor.png)

1. 只要preHandle不放行就没有以后的流程

2. 只要放行了，afterCompletion就会执行

3. 多个拦截器

   正常流程：

   ```
   谁先配置，谁先执行，prehandle方法，其余方法，反过来执行
   MyFirsInterceptor...preHandle...
   MySecondInterceptor...preHandle...
   test01
   MySecondInterceptor...postHandle...
   MyFirsInterceptor...postHandle...
   进入Success.jsp
   MySecondInterceptor...afterCompletion...
   MyFirsInterceptor...afterCompletion...
   ```

   异常流程：

   那块不放行，那里就不会执行了

   ```
   MySecondInterceptor不放行，但是前面已经放行的拦截器的afterCompletion总会放行
   MyFirsInterceptor...preHandle...
   MySecondInterceptor...preHandle...
   MyFirsInterceptor...afterCompletion...
   ```

拦截器运行流程

1. 拿到执行器链

![](resource\interceptor1.png)

2. 执行preHandle和postHandle方法

   ![](resource\interceptor2.png)

   ```java
   boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
   		HandlerInterceptor[] interceptors = getInterceptors();
   		if (!ObjectUtils.isEmpty(interceptors)) {
   			for (int i = 0; i < interceptors.length; i++) {
   				HandlerInterceptor interceptor = interceptors[i];
                   //一旦返回false就执行结束了
   				if (!interceptor.preHandle(request, response, this.handler)) {
                       //执行afterCompletion
   					triggerAfterCompletion(request, response, null);
                       这里返回false，上面的mappedHandler就直接返回了，下面的就不执行了
   					return false;
   				}
                   //记录已经执行的拦截器的索引
   				this.interceptorIndex = i;
   			}
   		}
   		return true;
   	}
   ```

   ```java
   void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
   			throws Exception {
   
   		HandlerInterceptor[] interceptors = getInterceptors();
   		if (!ObjectUtils.isEmpty(interceptors)) {
               //倒序执行
   			for (int i = interceptors.length - 1; i >= 0; i--) {
   				HandlerInterceptor interceptor = interceptors[i];
   				interceptor.postHandle(request, response, this.handler, mv);
   			}
   		}
   	}
   ```

   ```java
   private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
   			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
   			@Nullable Exception exception) throws Exception {
   
   		boolean errorView = false;
   
   		if (exception != null) {
   			if (exception instanceof ModelAndViewDefiningException) {
   				logger.debug("ModelAndViewDefiningException encountered", exception);
   				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
   			}
   			else {
   				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
   				mv = processHandlerException(request, response, handler, exception);
   				errorView = (mv != null);
   			}
   		}
   
   		// Did the handler return a view to render?
   		if (mv != null && !mv.wasCleared()) {
               //页面渲染
   			render(mv, request, response);
   			if (errorView) {
   				WebUtils.clearErrorRequestAttributes(request);
   			}
   		}
   		else {
   			if (logger.isTraceEnabled()) {
   				logger.trace("No view rendering, null ModelAndView returned.");
   			}
   		}
   
   		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
   			// Concurrent handling started during a forward
   			return;
   		}
   
   		if (mappedHandler != null) {
   			// 执行AfterCompletion，即使没走到这，afterrCompletion也会执行
   			mappedHandler.triggerAfterCompletion(request, response, null);
   		}
   	}
   ```

   ```java
   	void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
   			throws Exception {
   
   		HandlerInterceptor[] interceptors = getInterceptors();
   		if (!ObjectUtils.isEmpty(interceptors)) {
               //从最后一个执行的拦截器开始执行方法
   			for (int i = this.interceptorIndex; i >= 0; i--) {
   				HandlerInterceptor interceptor = interceptors[i];
   				try {
   					interceptor.afterCompletion(request, response, this.handler, ex);
   				}
   				catch (Throwable ex2) {
   					logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
   				}
   			}
   		}
   	}
   ```

# SpringMVC异常处理

![](resource\exceptionResolver.png)

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;

        //如果有异常
		if (exception != null) {
            //判断是不是ModeAndViewDefiningException异常
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
                //处理别的异常
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
            //视图渲染
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}

		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}

		if (mappedHandler != null) {
			// Exception (if any) is already handled..
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}

```

```java
protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
			@Nullable Object handler, Exception ex) throws Exception {

		// Success and error responses may use different content types
		request.removeAttribute(HandlerMapping.PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE);

		// 调用默认的异常处理器进行处理，处理不了直接返回异常，交给Tomcat处理
		ModelAndView exMv = null;
		if (this.handlerExceptionResolvers != null) {
			for (HandlerExceptionResolver resolver : this.handlerExceptionResolvers) {
				exMv = resolver.resolveException(request, response, handler, ex);
				if (exMv != null) {
					break;
				}
			}
		}
		if (exMv != null) {
			if (exMv.isEmpty()) {
				request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
				return null;
			}
			// We might still need view name translation for a plain error model...
			if (!exMv.hasView()) {
				String defaultViewName = getDefaultViewName(request);
				if (defaultViewName != null) {
					exMv.setViewName(defaultViewName);
				}
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Using resolved error view: " + exMv, ex);
			}
			else if (logger.isDebugEnabled()) {
				logger.debug("Using resolved error view: " + exMv);
			}
			WebUtils.exposeErrorRequestAttributes(request, ex, getServletName());
			return exMv;
		}

		throw ex;
	}
```

- ExceptionHandlerExceptionResolver:@ExceptionHandler

  ```java
   @org.springframework.web.bind.annotation.ExceptionHandler(ArithmeticException.class)
      public ModelAndView handleException(Exception e){
          System.out.println(e);
          System.out.println("错误页面");
          ModelAndView mv = new ModelAndView("error");
          mv.addObject("ex",e);
          return mv;
      }
  ```

- ResponseStatusExceptionResolver:@ResponseStatus

  给自定义异常标注的

  ```java
      @RequestMapping("/handleException2")
      public String handle02(@RequestParam("username") String username){
          if(!"admin".equals(username)){
              System.out.println("登录失败");
              throw new UserNameNotFoundException();
          }
          return "success";
      }
  
  @ResponseStatus(reason = "用户登录拒接",value = HttpStatus.NOT_ACCEPTABLE)
  class UserNameNotFoundException extends RuntimeException{
  }
  ```

  ![](resource\error.png)

- DefaultHandlerExceptionResolver:判断是否是SpringMVC自带异常

  ![](resource\error1.png)

```java
	protected ModelAndView doResolveException(
			HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

		try {
            //可以处理的SpringMVC自带异常
			if (ex instanceof HttpRequestMethodNotSupportedException) {
				return handleHttpRequestMethodNotSupported(
						(HttpRequestMethodNotSupportedException) ex, request, response, handler);
			}
			else if (ex instanceof HttpMediaTypeNotSupportedException) {
				return handleHttpMediaTypeNotSupported(
						(HttpMediaTypeNotSupportedException) ex, request, response, handler);
			}
			else if (ex instanceof HttpMediaTypeNotAcceptableException) {
				return handleHttpMediaTypeNotAcceptable(
						(HttpMediaTypeNotAcceptableException) ex, request, response, handler);
			}
			else if (ex instanceof MissingPathVariableException) {
				return handleMissingPathVariable(
						(MissingPathVariableException) ex, request, response, handler);
			}
			else if (ex instanceof MissingServletRequestParameterException) {
				return handleMissingServletRequestParameter(
						(MissingServletRequestParameterException) ex, request, response, handler);
			}
			else if (ex instanceof ServletRequestBindingException) {
				return handleServletRequestBindingException(
						(ServletRequestBindingException) ex, request, response, handler);
			}
			else if (ex instanceof ConversionNotSupportedException) {
				return handleConversionNotSupported(
						(ConversionNotSupportedException) ex, request, response, handler);
			}
			else if (ex instanceof TypeMismatchException) {
				return handleTypeMismatch(
						(TypeMismatchException) ex, request, response, handler);
			}
			else if (ex instanceof HttpMessageNotReadableException) {
				return handleHttpMessageNotReadable(
						(HttpMessageNotReadableException) ex, request, response, handler);
			}
			else if (ex instanceof HttpMessageNotWritableException) {
				return handleHttpMessageNotWritable(
						(HttpMessageNotWritableException) ex, request, response, handler);
			}
			else if (ex instanceof MethodArgumentNotValidException) {
				return handleMethodArgumentNotValidException(
						(MethodArgumentNotValidException) ex, request, response, handler);
			}
			else if (ex instanceof MissingServletRequestPartException) {
				return handleMissingServletRequestPartException(
						(MissingServletRequestPartException) ex, request, response, handler);
			}
			else if (ex instanceof BindException) {
				return handleBindException((BindException) ex, request, response, handler);
			}
			else if (ex instanceof NoHandlerFoundException) {
				return handleNoHandlerFoundException(
						(NoHandlerFoundException) ex, request, response, handler);
			}
			else if (ex instanceof AsyncRequestTimeoutException) {
				return handleAsyncRequestTimeoutException(
						(AsyncRequestTimeoutException) ex, request, response, handler);
			}
		}
		catch (Exception handlerEx) {
			if (logger.isWarnEnabled()) {
				logger.warn("Failure while trying to resolve exception [" + ex.getClass().getName() + "]", handlerEx);
			}
		}
		return null;
	}
```

- SimpleMappingExceptionResolve:通过配置的方式进行异常处理

  ```xml
      <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
          <property name="exceptionMappings">
              <props>
                  处理异常的类型，                            去的页面
                  <prop key="java.lang.NullPointerException">error</prop>
              </props>
          </property>
      </bean>
  ```

  
# HelloWorld运行细节

## 1.运行流程：

-  客户端点击链接会发送http://localhost：8080/springmvc/hello
- 来到Tomcat服务器
- SpringMVC收到前端控制器的所有请求
- 来看请求地址和@RequestMapping标注的那个匹配，来找到到底使用那个类的那个方法
- 方法执行完成之后会有一个返回值，SpringMVC认为这个返回值就是要去的页面地址
- 拿到方法的返回值之后，用视图解析器进行拼串得到完整的页面地址。
- 拿到页面地址，前端控制器帮我们转发页面。

## 2.RequestMapping

- 告诉SpringMVC这个方法用来处理什么请求

- /可以省略，即使省略了，也是默认从当前项目下开始。习惯加上比较好

## 3.不指定spring的配置文件位置

<img src="resource\默认文件.png" style="zoom: 50%;" />

## 4./和/*的区别

- /：拦截所有请求，不拦截jsp页面，*.jsp请求

- /*：拦截所有请求，拦截jsp页面，*.jsp请求

- *.jsp是tomcat的做的事

- 服务器中的大web.xml中有一个DefaultServlet，它的是url-pattern=/,是Tomcat中处理静态资源的（除了jsp和servlet之外的都是静态资源），前端控制器中的/禁用了tomcat中的DefaultServlet，

- 前端控制器url-pattern=/，静态资源例如index.html来到dispatcherServlet看到这个方法的RequestMapping是这个index.html,所以不能访问了
- 服务器中有默认的JspServlet的配置，所以可以访问*.jsp

# RequestMapping

## method属性

<img src="resource\method方法.png" style="zoom: 80%;" />

指定请求的方式，4xx都是客户端的错误，客户端的请求方式不对，路径不对。

## params属性

```java
    必须指定请求参数里面包含username属性
    @RequestMapping(value = "/handle03",params = {"username"})
	必须指定请求参数里面不带username属性
    @RequestMapping(value = "/handle03",params = {"!username"})
```

## heders属性

和params一样，可以指定参数，可以指定浏览器访问

## consumes属性

只接受内容类型是那种的请求，规定请求头中的Content-Type

## procuces属性

告诉浏览器发挥类型是什么，给响应的头上加上Content-Type:text/html,charset=utf-8

## Ant风格的模糊匹配

- ？：代替任意一个字符

- *：任意多个字符，和一层路径

- **：代替多层路径

  模糊和精确匹配的时候，精确优先

```
"/handle0?"，可以匹配handle01,handle02,...handle09
```

# Rest

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

# SpringMVC如何获取请请求信息

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

# 处理乱码

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

# 给页面输出数据

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

# ModeAttribute

- SpringMVC要封装请求参数的Book对象不应该是自己new出来的。而应该是【从数据库中】拿到准备好的对象，再来使用这个对象封装请求参数

- 方法位置：这个方法会提前于目标方法先运行。可以提前查出数据库中图书的信息。
- 参数位置：可以获取对象。
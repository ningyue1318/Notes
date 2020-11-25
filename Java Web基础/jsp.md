

# jsp

## jsp定义

- jsp全称是Java server Pages,java的服务器页面

- 主要作用是代替Servlet程序回传html页面

- jsp页面本质上是一个servlet程序，第一次访问的时候会被编译为HttpJspPage类，间接的继承了HttpServlet类，第一次访问hello.jsp时候，会生成hello_jsp.java文件

  _jspInit()             初始化

  _jspDestroy()     销毁

  _jspService         服务

  ```java
  public final class hello_jsp extends org.apache.jasper.runtime.HttpJspBase
      implements org.apache.jasper.runtime.JspSourceDependent,
                   org.apache.jasper.runtime.JspSourceImports {
  
    private static final javax.servlet.jsp.JspFactory _jspxFactory =
            javax.servlet.jsp.JspFactory.getDefaultFactory();
  
    private static java.util.Map<java.lang.String,java.lang.Long> _jspx_dependants;
  
    private static final java.util.Set<java.lang.String> _jspx_imports_packages;
  
    private static final java.util.Set<java.lang.String> _jspx_imports_classes;
  
    static {
      _jspx_imports_packages = new java.util.HashSet<>();
      _jspx_imports_packages.add("javax.servlet");
      _jspx_imports_packages.add("javax.servlet.http");
      _jspx_imports_packages.add("javax.servlet.jsp");
      _jspx_imports_classes = null;
    }
  
    private volatile javax.el.ExpressionFactory _el_expressionfactory;
    private volatile org.apache.tomcat.InstanceManager _jsp_instancemanager;
  
    public java.util.Map<java.lang.String,java.lang.Long> getDependants() {
      return _jspx_dependants;
    }
  
    public java.util.Set<java.lang.String> getPackageImports() {
      return _jspx_imports_packages;
    }
  
    public java.util.Set<java.lang.String> getClassImports() {
      return _jspx_imports_classes;
    }
  
    public javax.el.ExpressionFactory _jsp_getExpressionFactory() {
      if (_el_expressionfactory == null) {
        synchronized (this) {
          if (_el_expressionfactory == null) {
            _el_expressionfactory = _jspxFactory.getJspApplicationContext(getServletConfig().getServletContext()).getExpressionFactory();
          }
        }
      }
      return _el_expressionfactory;
    }
  
    public org.apache.tomcat.InstanceManager _jsp_getInstanceManager() {
      if (_jsp_instancemanager == null) {
        synchronized (this) {
          if (_jsp_instancemanager == null) {
            _jsp_instancemanager = org.apache.jasper.runtime.InstanceManagerFactory.getInstanceManager(getServletConfig());
          }
        }
      }
      return _jsp_instancemanager;
    }
  
    public void _jspInit() {
    }
  
    public void _jspDestroy() {
    }
  
    public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
        throws java.io.IOException, javax.servlet.ServletException {
  
      final java.lang.String _jspx_method = request.getMethod();
      if (!"GET".equals(_jspx_method) && !"POST".equals(_jspx_method) && !"HEAD".equals(_jspx_method) && !javax.servlet.DispatcherType.ERROR.equals(request.getDispatcherType())) {
        response.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, "JSPs only permit GET POST or HEAD");
        return;
      }
  
      final javax.servlet.jsp.PageContext pageContext;
      javax.servlet.http.HttpSession session = null;
      final javax.servlet.ServletContext application;
      final javax.servlet.ServletConfig config;
      javax.servlet.jsp.JspWriter out = null;
      final java.lang.Object page = this;
      javax.servlet.jsp.JspWriter _jspx_out = null;
      javax.servlet.jsp.PageContext _jspx_page_context = null;
  
  
      try {
        response.setContentType("text/html;charset=UTF-8");
        pageContext = _jspxFactory.getPageContext(this, request, response,
        			null, true, 8192, true);
        _jspx_page_context = pageContext;
        application = pageContext.getServletContext();
        config = pageContext.getServletConfig();
        session = pageContext.getSession();
        out = pageContext.getOut();
        _jspx_out = out;
  
        out.write("\r\n");
        out.write("\r\n");
        out.write("<html>\r\n");
        out.write("<head>\r\n");
        out.write("    <title>Title</title>\r\n");
        out.write("</head>\r\n");
        out.write("<body>\r\n");
        out.write("<h1>Hello.jsp</h1>\r\n");
        out.write("<a href=\"/\">挑战back</a>\r\n");
        out.write("</body>\r\n");
        out.write("</html>\r\n");
      } catch (java.lang.Throwable t) {
        if (!(t instanceof javax.servlet.jsp.SkipPageException)){
          out = _jspx_out;
          if (out != null && out.getBufferSize() != 0)
            try {
              if (response.isCommitted()) {
                out.flush();
              } else {
                out.clearBuffer();
              }
            } catch (java.io.IOException e) {}
          if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
          else throw new ServletException(t);
        }
      } finally {
        _jspxFactory.releasePageContext(_jspx_page_context);
      }
    }
  }
  
  ```

## jsp三种语法

### page指令

修改jsp页面的重要属性或者行为

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

|   language属性   | 表示jsp翻译后是什么语言的文件，暂时只支持java                |
| :--------------: | :----------------------------------------------------------- |
| contentType属性  | 表示jsp返回的数据类型是什么，源码中response.setContentType("text/html;charset=UTF-8"); |
| pageEncoding属性 | 表示当前jsp页面本身的字符集                                  |
|    import属性    | 跟java源代码一样导包导类                                     |
|  autoFlush属性   | out输出流使用 设置缓冲区满了，是否自动刷新                   |
|    buffer属性    | out输出流使用 设置缓冲区的大小                               |
|  errorPage属性   | 设置jsp页面运行时出错，自动跳转页面                          |
|   session属性    | 设置访问当前jsp页面，是否会创建HttpSession对象，默认是true   |
|   extends属性    | 设置jsp翻译出来的java类默认继承的类                          |

### jsp常用脚本

- 声明脚本，可以给jsp翻译出来的java类定义属性和方法，甚至静态代码块

  ```java
  <%! 声明java代码 %>
  <%!
  	private Interger id;
  %>
  ```

- 表达式脚本，在jsp页面上输出数据。所有的表达式脚本都会被翻译到_jspService()中的out.print()方法中

  ```
  <%= 表达式%>
  ```

  ![](resource\jsp表达式.png)

- 代码脚本，可以在jsp页面中编写自己需要的功能

  ```
  <% java代码 %>
  ```

### jsp注释

```
<!-- html 注释-->
<%-- jsp 注释 --%>
<%
    //java注释
%> 
```

## jsp九大内置对象

jsp中的内置对象，是指Tomcat在翻译jsp页面成为servlet源代码之后，内部提供的九大对象，叫内置对象。

![](resource\jsp九大对象.png)

- out对象主要设置缓存大小，是否自动刷新
- request和response是servlet的对象
- config是ServletConfig
- session是HttpSession
- application是ServletContext对象
- page就是当前编译后的jsp页面
- exception是java中的Exception对象
- pageContext是jsp页面编译后的内容，可以获取8个内置对象

 ![](resource\pageContext.png)

## jsp四大域对象

| pageContext | PageContextImpl类    | 当前jsp页面内有效                                          |
| ----------- | -------------------- | ---------------------------------------------------------- |
| request     | HttpServletRequest类 | 一次请求内有效                                             |
| session     | HttpSession类        | 一个会话范围内有效（打开浏览器访问服务器，直到浏览器关闭） |
| application | ServletContext类     | 整个web工程内都有效（只要web工程不停止）                   |

作用域从小到大

pageContext->request->session->application

## jsp常用标签

```jsp
静态包含
	<%@ include file =""%>
动态包含
	<jsp:include page="" >
请求转发
    <jsp:forward page="">
```

# EL

- EL表达式的全称是：Expression Language表达式语言。就是用下面的方式输出：

  ![](resource\EL.png)

- EL表达式的主要作用是替代jsp页面中的表达式脚本在jsp页面中进行数据的输出。

- EL输出的时候，要比jsp的表达式脚本要简洁的多。

- 如果EL找不到所需要的属性，返回的是“”，而不是NULL

|       变量       |         类型         |                 作用                 |
| :--------------: | :------------------: | :----------------------------------: |
|   pageContext    |   PageContextImpl    |       获取jsp中的九大内置对象        |
|    pageScope     |  Map<String,Object>  |      获取pageContext域中的数据       |
|   requestScope   |  Map<String,Object>  |        获取request域中的数据         |
|   sessionScope   |  Map<String,Object>  |         获取session中的数据          |
| applicationScope |  Map<String,Object>  |     获取servletContext域中的数据     |
|      param       |  Map<String,Object>  |              请求参数值              |
|   paramValues    | Map<String,String[]> |            多个请求参数值            |
|      header      |  Map<String,String>  |              请求头信息              |
|   headerValues   | Map<String,String[]> |            多个请求头信息            |
|      cookie      |  Map<String,Cookie>  |            当前cookie信息            |
|    initParam     |  Map<String,String>  | web.xml中的<context-param>上下文参数 |

  ![](resource\EL2.png)

# JSTL

- JSTL标签库，全称是JSP Standard Tag Library .JSP标准标签库，是一个不断完善的开放源代码的JSP标签库。

- EL表达式主要是为了替换jsp中的表达式脚本，而JSTL是为了替换脚本代码，这样整个jsp页面变得更加简洁。

  ```jsp
  导入JSTL库
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  ```

### c:set

  ![](resource\jstl.png)

  
# Servlet

## Servlet生命周期

```java
public class MyServlet implements Servlet {
    @Override
    public void init(ServletConfig config) throws ServletException {
        System.out.println("初始化"+new Date());
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        System.out.println("执行了服务方法"+new Date());
        res.getWriter().write("Helloworld");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("销毁方法执行"+new Date());
    }
}
```

![](resource\servlet生命周期.png)

## Web目录

- /如果被浏览器解析，得到的地址是http://ip:port/

  ```
  <a href="/"></a>
  response.sendRediect("/")
  ```

- /如果被服务器解析，得到的地址是：http://ip:port/工程路径

  ```
  <url-pattern>/servlet1</utrl-pattern>
  servletContext.getRealPath("/")
  request.getRequestDispatcher("/")
  ```

## ServletConfig

获取web.xml中servlet标签下的init-param的值

![](resource\ServletConfig.png)

## ServletContext

- 当Tomcat启动的时候，创建一个ServletContext对象，代表当前web站点。
- 所有的Servlet都共享一个ServletContext对象，

![](resource\ServletContext.png)

## Cookie

- Cookie是服务器通知客户端保存键值对的一种技术。
- 客户端有了Cookie后，每次请求都发送给服务器。
- 每个cookie的大小不能超过4kb

### Cookie的创建

```java
     Cookie cookie = new Cookie("key1","value1");
     resp.addCookie(cookie);
     resp.getWriter().write("Cookie create is ok");
```

![](resource\Cookie.png)

### Cookie获取

```
req.getCookies()//获取服务器客户端的Cookie
```

### Cookie生命控制

```
cookie.setMaxAge(-1)//设置存活时间，为session,浏览器一关，立马删除
```

## Session

- session是一个接口，会话，用来维护客户端和服务器关联的一种技术。
- 每个客户端都有自己的一个Session会话。
- 用来保存用户登录后的信息。

```java
//第一次调用创建Session会话，之后调用都是获取前面创建好的Session会话。
request.getSession()
```

![](resource\session.png)

### Session和Cookie的区别

![](resource\Cookie&Session.png)

# Filter

- Filter过滤器是JavaWeb三大组件之一，三大组件分别是Servlet程序、Listener监听器、Filter过滤器。
- Filter是JAVAEE规范，也就是接口，作用是拦截请求，过滤响应。
- 应用场景，权限检查，日记操作，事物管理

# Filter&AOP&Interceptor区别

- Filter过滤器：拦截Web访问URL地址，是Servlet规范，基于职责链
- Interceptor:SpringMVC中概念，基于代理。
- 两者都是AOP的实现
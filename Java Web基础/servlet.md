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
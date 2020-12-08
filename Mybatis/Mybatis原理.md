Mybatis运行流程

<img src="resource\1自定义Mybatis分析.png" style="zoom:150%;" />

<img src="resource\2查询所有的分析.png" style="zoom:150%;" />

<img src="resource\3mybatis的分析.png" style="zoom:150%;" />

自定义Mybatis流程图

<img src="resource\自定义mybatis开发流程图.png" style="zoom:150%;" />

## Mybatis运行流程

1. sqlSession.getMapper()

<img src="resource\原理1.png" style="zoom:80%;" />

2. DefaultSqlSession#getMapper函数

<img src="resource\原理2.png" style="zoom:80%;" />

3. Configuration#getMapper函数

<img src="resource\原理3.png" style="zoom:80%;" />

4. MapperRegistry#getMeaaper函数

<img src="resource\原理4.png" style="zoom:80%;" />

5. MapperProxyFactory#newInstance函数,到此创建代理对象完成，下面开始函数的执行

   <img src="resource\原理5.png" style="zoom:80%;" />

6. userDao.findAll(),进入MapperProxy#invoke方法

   <img src="resource\原理6.png" style="zoom:80%;" />

7. MapperMethod#execute方法

   <img src="resource\原理7.png" style="zoom:80%;" />

8. MapperMethod#executeForMany方法

   <img src="resource\原理8.png" style="zoom:80%;" />

9. DefaultSqlSession#selectList方法

   <img src="resource\原理9.png" style="zoom:80%;" />

10. CachingExecutor#query方法

    <img src="resource\原理10.png" style="zoom:80%;" />

11. BaseExecutor#query方法

    <img src="resource\原理11.png" style="zoom:80%;" />



12.RoutingStatementHandler#query方法，这里的PreparedStatement是JDBC原生方法

<img src="resource\原理12.png" style="zoom:80%;" />
































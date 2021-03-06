# Mybatis

- MyBatis:和数据库交互，持久层框架(SQL映射框架)

- 从原始的JDBC-dbutils(QueryRunner)-JdbcTemplate-xxx被称为工具

  工具：一些功能的简单封装

  框架：某个领域的整体解决方案，缓存，考虑异常处理问题，考虑部分字段映射问题

- Hibernate-数据库交互的框架(ORM框架)，做的是对象关系映射，只需要操作对象，框架负责和数据库的交互。只需要写一个JavaBean就可以和数据库的表进行关联。

  缺点：定制sql困难

## Mybatis入门

1. 导包

```xml
 <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
```

2. 编写全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/book"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="EmployeeDao.xml"/>
    </mappers>
</configuration>
```

3. 编写SQL映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.syn.dao.EmployeeDao">

    <select id="getEmpById" resultType="com.syn.bean.Employee">
        select * from t_employee where id = #{id}
    </select>
</mapper>
```

4. 测试

```java
         /*
            SqlSessionFactory,是SqlSession工厂，负责创建SqlSession对象
            SqlSession:sql会话，代表和数据库的一次会话
         */
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession openSession = sqlSessionFactory.openSession();
        EmployeeDao employeeDao = openSession.getMapper(EmployeeDao.class);
        Employee employee = employeeDao.getEmpById(1);
        System.out.println(employee);
```

## 全局配置文件

- 指导mybatis正确运行的一些全局配置

- SqlSessionFactory创建SqlSession对象

- configuration（配置）可以操作的对象

  - [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties)
  
  - [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings)
  
  - [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)
  
  - [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
  
  - [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
  
  - [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
  
  - environments（环境配置）
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
    
  - [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)
  
  - [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers)
  

### Properties标签

- 用来引用外部配置文件

  ```xml
      resource:用来从类路径加载，
      url：从磁盘路径加载
  	用${}来获取配置文件中的选项
  	<properties resource="dbconfig.properties"></properties>
  ```

### Setting标签

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等。

```xml
完整的Setting标签如下
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### typeAliases

类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
    <!--对于domai.blog下的所有类起别名，默认别名就是类名,也可以用@Alias()标签来指定包下面的别名-->
  <package name ="domain.blog"/>
</typeAliases>
```

### typeHandlers

MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

<img src="resource\typeHandler.png" style="zoom:80%;" />

### plugins

MyBatis 允许你在映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

上面是Mybatis的四大组件

### environments

```xml
       //每一个enviroment环境都需要配置两个东西，一个是transactionManage，一个是dataSource 
       //default指定默认环境
        <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/book"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
        </environments>
```

其中的type分为三种情况

- POOLED:采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现
- UNPOOLED:采用传统的获取连接的方式，虽然也实现了DataSource接口，但没有实现连接池的思想
- JNDI：采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器拿到的DataSource是不一样的，不是web或者maven的war工程，是不能使用的。Tomcat中的连接池是dbcp连接池

### databaseIdProvider

用来考虑数据库移植

### mapper

```xml
<mappers>
  类路径下寻找
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  磁盘文件寻找
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  直接引用接口全类名，将xml放在和dao接口同目录下，而且文件名和接口名一样
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
批量注册
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```



## SQL映射文件

- 相当于对DAO接口的一个实现描述
- 获取到的接口的代理对象
- SqlSession相当于connection和数据库进行交互，和数据库的一次会话，就应该创建一个新的SqlSession。

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。
- `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- `parameterMap` – 老式风格的参数映射。此元素已被废弃，并可能在将来被移除！请使用行内参数映射。文档中不会介绍此元素。
- `sql` – 可被其它语句引用的可重用语句块。
- `insert` – 映射插入语句。
- `update` – 映射更新语句。
- `delete` – 映射删除语句。
- `select` – 映射查询语句。

### delete,update,insert

```xml
<insert
  id="insertAuthor"                  
  parameterType="domain.blog.Author" 
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```

<img src="resource\insert&update&delete.png" style="zoom:80%;" />

- useGeneratedKeys和KeyProperty可以用来设置自增的ID

<img src="resource\自增ID.png" style="zoom:80%;" />

- 不支持自增的条件下，设置ID

```xml
    <insert id="insertEmployee2">
        提前运行方法，获取应该设置的id值
        <selectKey order="BEFORE" resultType="integer" keyProperty="id">
            select max(id)+1 from t_employee
        </selectKey>
        INSERT INTO t_employee(id,empname,gender,email) VALUES(#{id},#{empName},#{gender},#{email})
    </insert>
```

- 参数传入

  1. 传入单个参数，基本类型取值#{随便写}，

  2. 多个参数，#{参数名}无效，可用的是0,1或者param1，param2。原因只要传入了多个参数，MyBatis会将这些参数自动封装在map中，封装时使用的key就是参数的索引。

     即

     ```
     map.put("0",第一个参数)
     map.put("1",第二个参数)
     ```

  ​       如果要使用自己定义的参数需要使用@Param注解

  ```java
   public Employee getEmpByIdAndEmpName(@Param("id") Integer id, @Param("empName") String empName);
  ```

  <img src="resource\绑定错误异常.png" style="zoom:80%;" />

3. 传入pojo，取值#{pojo的属性名}
4. 传入map：将要使用的参数封装起来

- 两种取值方式的区别

  ```
  #{属性名}，是预编译的方式，参数的位置是用？替代，参数都是后来预编译设置进去的，安全不会有sql注入的方式
  ${属性名}，不是参数预编译，直接用sql语句进行拼串，不安全
  ```

### select

1. 查询结果返回一个List

   <img src="resource\返回值是List.png" style="zoom:80%;" />

2. 查询返回一个Map

   ​	返回单个map对象

<img src="resource\单个对象Map.png" style="zoom:80%;" />

​		返回多个对象

<img src="resource\多个对象的Map.png" style="zoom:80%;" />

3. 级联查询

   当属性里面包含对象的时候，采用级联查询来封装结果。

   ```xml
       <select id="getKeyById" resultMap="myKey">
         select k.id,k.keyname,k.lockid,l.id lid,l.lockName from t_key k LEFT JOIN t_lock l
          on k.id=l.id WHERE k.id=#{id}
       </select>
   
   
       <resultMap id="myKey" type="com.syn.bean.Key">
           <id property="id" column="id"/>
           <result property="keyName" column="keyname"/>
           <result property="lock.id" column="lid"/>
           <result property="lock.lockName" column="lockName"/>
       </resultMap>
   
   使用级联属性也可以封装，封装一个对象
   
       <resultMap id="myKey" type="com.syn.bean.Key">
           <id property="id" column="id"/>
           <result property="keyName" column="keyname"/>
           <association property="lock" javaType="com.syn.bean.Lock">
               <id property="id" column="ld"/>
               <result property="lockName" column="lockName"/>
           </association>
       </resultMap>
   ```

   <img src="resource\级联查询.png" style="zoom:80%;" />

   当属性里包含集合的时候，使用collection标签

   ```xml
       <select id="getLockById" resultMap="myLock">
           select k.id,k.keyname,k.lockid,l.id lid,l.lockName from t_key k LEFT JOIN t_lock l
          on k.lockid=l.id WHERE l.id=#{id}
       </select>
   
       <resultMap id="myLock" type="com.syn.bean.Lock">
           <id property="id" column="lid"/>
           <result property="lockName" column="lockName"/>
           <collection property="keys" ofType="com.syn.bean.Key">
               <id property="id" column="id"/>
               <result property="keyName" column="keyname"/>
           </collection>
       </resultMap>
   ```

4. 分步查询

   查询key表的lockid对应的lockname，连接查询

   <img src="resource\key-lock表.png" style="zoom:80%;" />

   <img src="resource\分步查询.png" style="zoom:80%;" />

### resultMap

```java
//User的命名规则，
public class User {
  private int id;
  private String username;
  private String hashedPassword;
}
//数据库中对应的列名分别为，user_id,user_name,hashed_password。为了让查询出来的结果自动封装进去，设置resultMap进行名称映射。
```

```xml
id属性指定主键，result指定其他列
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>

查询方式
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

## 动态SQL

### if标签

```xml
 <select id="getTeacherByCondition" resultMap="teacherMap">
          select * from t_teacher where
          <if test="id!=null">
              id>#{id} and
          </if>
        <if test="name!=null">
            teacherName like #{name} and
        </if>
        <if test="birth!=null">
            birth_date>#{birth}
        </if>
    </select>
```

### where标签

自动去除前面的and，如果条件不符合

### for-each标签

用来遍历集合，collections需要用@Param指定变量名，item是遍历的变量，separator是遍历的分隔符，open，close是开始关闭的标签

<img src="resource\for-each标签.png" style="zoom:80%;" />

```xml
   <select id="getTeacherByIdIn" resultMap="teacherMap">
        select * from t_teacher where id IN
        <foreach collection="ids" item="id_item" separator="," open="(" close=")">
            #{id_item}
        </foreach>
    </select>
```

### set和if标签动态更新

```xml
 <update id="updateTeacher">
        UPDATE t_teacher
        <set>
            <if test="name!=null">
                teacherName = #{name},
            </if>
            <if test="course!=null">
                class_name = #{course},
            </if>
            <if test="address!=null">
                address = #{address},
            </if>
            <if test="birth!=null">
                birth = #{birth}
            </if>
        </set>
        <where>
            id=#{id}
        </where>
    </update>
```

<img src="resource\Update.png" style="zoom:80%;" />

## 缓存

- 暂时的存储一些数据，加快系统的查询速度

- 一级缓存：线程级别的缓存，本地缓存，SqlSession级别的缓存

  只要之前查询过的数据，mybatis就会保存在一个缓存中（Map），下次获取直接从缓存中拿。

  <img src="resource\一级缓存.png" style="zoom:80%;" />

  缓存失效的几种情况

  1.  不同的sqlSession，使用不同的一级缓存。

  2. 同一个方法不同的参数，由于之前可能没查过，所以还会发新的sql

  3. 在sqlSession期间执行任何一次增删改操作，增删改操作会把缓存清空

  4. 清空当前sqlSession的一级缓存，clearCache()方法

  每次查询，先看一级缓存有没有，没有就发送新的sql，每个sqlSession拥有自己的缓存

- 二级缓存：全局范围的缓存，出国当前线程，SqlSession能用外也可以用。SqlSession关闭或者提交之后，一级缓存的数据会放在二级缓存中，默认没有使用，需要开启

  ```xml
  //开启二级缓存的开关
  <setting name="cacheEnabled" value="true"/>
  //配置某个dao.xml文件，在需要缓存的地方加上下面的话
  <cache></cache>
  ```

一级缓存和二级缓存不会同时出现一个数据

二级缓存中，一级缓存关闭就有了

二级缓存中没有数据，就会看一级缓存，一级缓存也没有，就会查询数据库


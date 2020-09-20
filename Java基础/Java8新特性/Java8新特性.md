# Java8新特性

## Lambda表达式

举例：（o1,o2）->Integer.compare(o1,o2);

格式：-> Lambda操作符 或 箭头操作符

​            ->左边：lambda形参列表（其实就是接口中抽象方法的形参列表）

​             ->右边：Lambda体，其实就是重写的抽象方法的方法体

Lambda作为接口的实例，接口内只有一个方法，所以不用提供名字

函数式接口：接口内只有一个抽象方法。

Lambda表达式的使用

- 无参 无返回值

  ```java
  Runnable r1 = ()-> System.out.println("输出"); ;
  ```

- 需要一个参数 但是没有函数值

  ```java
  Consumer<String> con =(String s) -> {
              System.out.println(s);
  };
  
  con.accept("你好");
  ```
  
- 数据类型可以省略，因为可由编译器推断出来

  ```java
  Consumer<String> con2 =(s) -> {
              System.out.println(s);
  };
  
  con2.accept("你好");
  ```
  
- Lambda只需要一个参数时，参数的小括号可以省略

  ```java
  Consumer<String> con2 =s -> {
              System.out.println(s);
  };
  
  con2.accept("你好");
  ```

- Lambda需要两个或以上的参数，多条执行语句，并且有返回值

  ```java
   Comparator<Integer> com2 = (o1,o2)->{
              System.out.println(o1);
              System.out.println(o2);
              return o1.compareTo(o2);
          };
  ```

- Lambda体只有一条语句，return与大括号要有，可以省略。

  ```java
   Comparator<Integer> com3 = (o1,o2)->o1.compareTo(o2);
  ```

### 函数式接口

Java内置的四大核心函数式接口

| 函数式接口    | 参数类型 | 返回类型 | 用途                               |
| ------------- | -------- | -------- | ---------------------------------- |
| Consumer<T>   | T        | void     | 为类型T的对象应用操作              |
| Supplier<T>   | 无       | T        | 返回类型为T的对象                  |
| Function<T,R> | T        | R        | 对类型T的对象应用操作，并返回结果R |
| Predicate<T>  | T        | boolean  | 确定类型T的对象是否满足条件        |

### 方法引用

当要传递给Lambda体的操作，已经有实现方法了，可以使用方法引用。分为以下三种情况

- 对象：：非静态方法

  ```java
  Consumer<String> con5 = System.out::println;
  con5.accept("测试方法引用");
  ```

- 类：：静态方法

  ```java
  Comparator<Integer> com5 = Integer::compare;
  System.out.println(com5.compare(2,3));
  ```

- 类：：非静态方法

  ```java
  //Comparator有两个参数，s1,s2,正常Comparator<String> com6 = (s1,s2)->s1.compareTo(s2)
  //compareTo有一个参数,s1.compareTo(s2)，可以写成这样的形式，就可以参数列表不匹配，两个变成了一个
  
  Comparator<String> com6 = String::compareTo;
  System.out.println(com6.compare("123","345"));
  ```

接口中的抽象方法的形参列表和返回值类型与方法引用的形参列表和返回值类型一致（针对一二两种情况）

## Stream API

使用Stream API可以对集合数据进行操作，类似使用SQL执行的数据库查询。Stream API 提供了一套高效且易于使用的处理数据的方式。

Stream API的优点：

- 实际开发中，项目中多数数据源都来自于MySQL，Oracle等，但是现在数据源更多了，如：MongDB，Redis等，而这些NoSQL的数据需要Java层面处理。
- Stream和Collection集合的区别：Collection是一种静态的内存数据结构，而Stream是有关计算的。前者主要面向内存，存储在内存中，后者主要面向CPU，通过CPU实现计算。

Stream操作的三个步骤：

- 创建Stream
- 中间操作
- 终止操作

### 创建Stream的方式：

- 通过集合创建

  ```java
  default Stream<E> stream(): 返回一个顺序流
  default Stream<E> parallelStream():返回一个并行流
  ```

- 通过数组创建

  ```java
  //Arrays的静态方法stream()可以获取数组流
  static <T> Stream<T> stream(T[] array):
  ```

- 通过Stream的of()

  ```java
  public static<T> Stream<T> of(T ...values):
  ```

### 中间操作

#### 筛选与切片

```java
//接收Lambda，从中筛选出某些元素，
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
list.add(4);
list.add(5);
Stream<Integer> stream = list.stream();
stream.filter(x ->x>2).forEach(System.out::println);

//截断流，只要执行完，下次执行前就要重新生成stream
list.stream().limit(3).forEach(System.out::println);

//跳过元素
skip()

//筛选,通过流所生成元素的hascode和equals来去除重复元素
distinct()
```

#### 映射

```java
//Map函数应用举例
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        list.stream()
                .map(x->x+2)
                .forEach(System.out::println);

```

#### 排序

```
sorted 排序
```

### 终止操作

#### 匹配与查找

```java
        List<Integer> list = new ArrayList<>();
        list.add(14343);
        list.add(23232);
        list.add(323);
        list.add(43);
        list.add(5);
        Object o =list.stream()
                .sorted()
                .max(Integer::compareTo);
        System.out.println(o);
```

#### 归约

```java
reduce(T identity,BinaryOperator) //可以将流中元素反复结合起来
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        Integer sum = list.stream()
                          .reduce(0,Integer::sum);
        System.out.println(sum);
```

#### 收集

```
List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        Object o = list.stream()
                .collect(Collectors.counting());
        System.out.println(o);
```

### Optional类

是一个容器类，它可以保存类型T的值，代表这个值存在，或者仅仅保存null，表示这个值不存在。原来用null表示一个值不存在，现在用Optional可以更好的表达这个概念，并且可以避免空指针异常。
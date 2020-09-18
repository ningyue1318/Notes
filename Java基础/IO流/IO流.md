# IO流

## IO

### IO流框架

- 按照操作数据单位不同分为：字节流（8bit）适合图片，视频等非文本形式、字符流（16bit）适合文本形式

- 按照数据的流向不同分为：输入流、输出流

| 抽象基类 |    字节流    | 字符流 |
| :------: | :----------: | :----: |
|  输入流  | InputStream  | Reader |
|  输出流  | OutputStream | Writer |

![](F:\学习资料\Nodes\Java基础\IO流\resources\IO框架图.jpg)

- 访问文件的是节点流，直接和文件进行操作的。剩下的都是处理流，用来对流进行包装的。
- 对应的列都是集成抽象基类。

### IO流用到的设计模式

- 装饰器设计模式 ，通过组合来代替继承，主要是给原始类增加功能，这是判断是不是装饰器的重要依据。另外装饰器可以对原始类套用多个装饰器。

  

为什么不设计一个继承FileInputStream并且支持缓存的BufferedFileInputStream类对象呢？这样直接创建BufferedFileInputStream就好了，为什么没有采用继承的方案呢，因为InputStream不是只有FileInputStream一个类，采用这样的方法的话，所有的子类都需要写一遍，增加了代码量，不如写一个接受InputStream的缓存类。

利用组合来代替继承的方案，解决继承关系过于复杂的问题



```java
//BufferInputStream的构造函数，可以看到接收了一个InputStream的对象，这样就可以只写一个类，避免继承重写多个类
public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
}

//DataInputStream也是这样
public DataInputStream(InputStream var1) {
        super(var1);
}
```

- 装饰器类和原始类继承同样的父类，这样可以对原始类嵌套多个装饰器类，例如BufferedInputStream和DataInputStream都继承FilterInputStream，而后者继承InputStream。

  ```java
  //这样既可以实现读取缓存，还可以按照基本类型来读取
  InputStream in = new FileInputStream("/user/test.txt")
  InputStream bin = new BufferedInputStream(in);
  DataInputStream din = new DataInputStream(bin);
  ```

- 装饰器对功能的增强，也是装饰器的应用场景的重要特点。

  ```java
  public interface IA{
      void f();
  }
  
  public class A implements IA{
      public void f(){};
  }
  
  public class ADecorator implements IA{
      private IA a;
      
      public ADecorator(IA a){
          this.a = a;
      }
      
      public void f(){
          //代码增强
          a.f();
      }
  }
  ```

  

### File

文件和文件目录路径的抽象表示，与平台无关，能新建、删除、重命名文件和目录，但File不能访问文件内容本身。

```java
File file = new File("hello.txt");   //相对路径，相当于当前module
```



### 字符流

FileReader的读取操作

```java
 File file = new File("D:\\IDEA\\StudySpace\\JavaSE\\src\\Io\\syn\\neu\\Hello.txt");
        FileReader fr = null;
        try {
            fr = new FileReader(file);
            
            int data;
            while((data=fr.read())!=-1){
                System.out.print((char)data);
            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                fr.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

FileWriter的写操作

```java
  File file = new File("D:\\IDEA\\StudySpace\\JavaSE\\src\\Io\\syn\\neu\\Hello1.txt");

        try {
            FileWriter fw = new FileWriter(file);
            fw.write("I have a dream\n");
            fw.write("you need to have a dream!");
            fw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {

        }
```

### 字节流

和字符流的操作类似，只是对应的类换成FileInputStream和FileOutputStream



### 缓冲流

提高流的读取和写入的速度，内部提供了一个缓冲区

```
File file = new File("D:\\IDEA\\StudySpace\\JavaSE\\src\\Io\\syn\\neu\\Hello.txt");
        FileInputStream fio= null;
        BufferedInputStream bis = null;

        try {
            fio = new FileInputStream(file);
            bis = new BufferedInputStream(fio);

            int len;
            byte []buffer = new byte[5];
            while((len = bis.read(buffer))!=-1){
                String str = new String(buffer,0,len);
                System.out.println(str);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                bis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                fio.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

转换流：把字节流转换为字符流



### 标准的输入，输出流

- System.in 标准输入流，默认从键盘输入

- System.out 标准输出流，默认从控制台输出


通过System类的setIn（）/setout()方式重新制定输入和输出的流。



### 打印流

- PrintStream和PrintWriter

提供了一些列print()和println()的方法，用来输出数据。

具有自动flush的功能，输出不会抛出IOException异常

System.out返回的是PrintStream的实例



### 数据流

DataInputStream和DataOutputStream两个类，用来操作基本数据类型和String。



## 字符集

常见的编码表

- ASCII：美国标准信息交换码。用一个字节的7位表示。

- ISO8859-1：拉丁码表，欧洲码表，用一个字节的8位表示。

- GB2312：中国中文编码表，最多两个字节编码所有的字符

- GBK：GB2312的升级融入了更多的字符，最多两个字节

上面都是各个国家的码表，为了融合，产生了Unicode编码

- Unicode：国际标准码，融合了人类使用的所有字符，为每个字符分配一个唯一的字符吗，所有的文字都用两个字节表示
- UTF-8：变长的编码方式，可用1-4个字节来表示一个字符。是对Unicode的精简。


# HTML

## HTML书写规范

```html
<!DOCTYPE html>               <!--约束，声明-->          
<html lang="en">              <!--html标签标示html开始，lang="zh_CN"表示中文，
                                     html标签分为两部分，分别是head和body  -->
<head>                        <!-- 表示头部信息，一般分为三部分，title标签，css样式，js代码-->
    <meta charset="UTF-8">    <!--表示当前字符编码-->
    <title>Title</title>
</head>
<body>                        <!--主体内容-->
你好啊
</body>
</html>
```

## HTML常用标签

### < a>标签

```html
<!--
    <a>超链接
    href超链接：属性设置连接的地址
    target属性设置那个目标进行跳转
            _self   表示当前页面
            _blank  表示要跳转的页面
-->
<a href="target.html">百度一下</a>
<a href="target.html" target="_self">百度一下</a>
<a href="target.html" target="_blank">百度一下</a>
```

### < img>标签

```html
JavaSE中相对路径从工程名算，绝对路径从盘符开始算
web中
	相对路径：
		.	当前文件所在目录
        ..  当前文件所在的上一级目录
        文件名 当前文件所在目录文件，相当于./文件名
	绝对路径：
		正确格式：http://ip:port/工程名/资源路径

<!--
    src:设置图片的路径
    width:设置图片的宽度
    height:设置图片的高度
    border:设置图片的边框大小
    alt：当指定路径找不到图片的时候，用来代替文本内容
-->
    <img src="img/8.jpg" width="270" height="408" border="20" alt="图片没了！"/>
```

### 表单标签

- action设置提交的地址
- method设置提交的方式
- 表单提交的时候，数据没有发给服务器的三种情况:
  1. 表单项没有name属性值
  2. 单选，复选都需要添加value属性，以便发送给服务器
  3. 表单项不再提交的form标签中
- GET请求的特点：

  1. 浏览器地址栏中的地址是：action属性[+?+请求参数]

  2. 不安全

  3. 有数据长度限制
- POST请求的特点：
  1. 浏览器中只有action属性值
  2. 相对于GET请求更安全
  3. 理论上没有数据长度的限制

```html
<form>
    <table align="center">
        <tr>
            <td> 用户名称：</td>
            <td><input type="text" value="默认值"></td>
        </tr>
        <tr>
            <td> 用户密码：</td>
            <td><input type="password"></td>
        </tr>
        <tr>
            <td> 确认密码:</td>
            <td><input type="password"></td>
        </tr>
        <tr>
            <td> 性别：</td>
            <td><input type="radio" name="sex" checked="checked">男<input type="radio" name="sex">女</td>
        </tr>
        <tr>
            <td>  兴趣爱好：</td>
            <td><input type="checkbox">Java<input type="checkbox">JavaScript<input type="checkbox">C++</td>
        </tr>
        <tr>
            <td>  国籍:</td>
            <td><select>
                <option selected="selected">中国</option>
            </select></td>
        </tr>
        <tr>
            <td>  自我评价:</td>
            <td><textarea rows="10" cols="20"></textarea></td>
        </tr>
    </table>
</form>
```

### 其他标签

- div标签，默认独占一行
- span标签，长度是封装数据的长度
- p段落标签，默认会在段落上方或者下方空出一行来

# CSS

## CSS和HTML结合方式

1. 直接在标签style属性上设置

   <img src="resource\html1.png" style="zoom:80%;" />

2. 在head标签中，使用style标签定义自己需要的css样式（同一页面复用）

   <img src="resource\html2.png" style="zoom:80%;" />

3. 把css标签写成一个单独的css文件，在通过link引入（所有文件复用）

   <img src="resource\html3.png" style="zoom:80%;" />

## 选择器

- 标签名选择器

  ```html
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style type="text/css">
          div{
              border: 1px solid red;
          }
      </style>
  </head>
  <body>
      <div>div标签</div>
  
  
  </body>
  ```

- id选择器

  ```html
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style type="text/css">
          #id001{
              color:blue;
              border: 1px solid red;
              border: 1px yellow solid;
          }
      </style>
  </head>
  <body>
      <div id="id001">div标签</div>
  </body>
  ```

- class选择器(类选择器)

  ```html
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style type="text/css">
          .class01{
              color:blue;
              border: 1px solid red;
              border: 1px yellow solid;
          }
      </style>
  </head>
  <body>
      <div class="class01">div标签</div>
  </body>
  ```

- 组合选择器

  ```html
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style type="text/css">
          .class01,#id01{
              color:blue;
              border: 1px solid red;
              border: 1px yellow solid;
          }
      </style>
  </head>
  <body>
      <div class="class01">div标签</div>
  </body>
  ```

  
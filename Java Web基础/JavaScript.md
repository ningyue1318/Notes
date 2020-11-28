# JavaScript

## 变量类型

- 数值变量类型   number
- 字符串类型 string
- 对象类型 object
- 布尔类型 boolean
- 函数类型 function
- 特殊类型
  - undefined 未定义，所有js变量未赋值的时候，默认值都是这个
  - null，空值
  - NAN 全称是Not a Number 非数字，非数值

var 变量名 

# jQuery

## jQuery基础

- jQuery，是JavaScript和查询，辅助JavaScript开发的类库

- 核心思想是write less,do more（写的更少，做的更多），所以实现了很多浏览器的兼容问题。

- jQuery中的$是一个函数。

  ![](resource\jQuery.png)

  1. 传入参数为函数时，表示页面加载完成之后，相当于window.onload = funcation(){}

  2. 传入参数为HTML字符串的时候，会对我们这个HTML标签创建对象

  3. 闯入参数为选择器字符串的时候：

     $("#id属性值") id选择器，根据id值查询标签对象

     $("标签名")标签名选择器，根据指定的标签名查询标签对象

     $(".class属性值")类型选择器，可以根据class属性查询标签对象
  4. 传入参数是DOM对象，会把这个dom对象转化为jQuery对象

## 选择器

[JQuery文档]: jQuery_api\jQueryAPI_1.7.1_CN.chm


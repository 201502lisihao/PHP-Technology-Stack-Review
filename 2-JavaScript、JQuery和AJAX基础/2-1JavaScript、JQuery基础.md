## 2.1 JavaScript、JQuery和AJAX基础 --  JavaScript、JQuery
***

### JavaScript基础语法

**变量的定义：**

> 1. 变量必须以字母、 $ 或 _下划线开头。
> 2. 变量对大小写敏感。
> 3. 使用var关键字定义变量。
> 4. 一条语句中声明很多变量，如 var a=1, b=2, c=3;
> 5. 未使用值来声明的对象，值是undefined，如 var a;
> 6. 如果重新声明变量，变量的值不会丢失，如 var a=1; var a;

**数据类型：**

> 字符串、数字、布尔、数组、对象、Null、Undefined

`tip：JavaScript的变量均为对象，当你声明一个变量时，就创建了一个新的对象。`

**对象：**

> 1. 使用new Object()方式声明。
> 2. 使用对象构造器，一个function里写this.xxx，使用时new这个function即可。
> 3. JSON对象直接使用{}花括号定义。

**函数：**

> 1. 函数使用function定义。
> 2. 形参不支持默认值。
> 3. 函数内部声明的变量（使用var）是局部变量。
> 4. 在函数外部声明的变量是全局变量，所有的脚本和函数都能访问它。

**运算符：**

> 注意：`+`可以用来做字符串拼接，PHP中使用 `.` 来拼接字符串。

**流程控制：**

> 注意：`else if`必须分开写，PHP中`elseif`可以连着写也可以分开写。

### JavaScript内置对象

**Nubmer：**
```javascript
// 方式1
var pi = 3.14;
// 方式2
var number = new Number(123);
// 方式3
var number = Number(123);
```

**String：**
```javascript
// 方式1，单引号和双引号，在js里几乎没有区别
var str1 = 'test1';
var str2 = "test2";
// 方式2
var str = new String('test');
// 方式3
var str = String('test'); 
```
拓展：[String的方法和属性](https://www.w3school.com.cn/js/js_string_methods.asp)

**Boolean：**
```javascript
// 方式1
var bool = true;
// 方式2
var bool = new Boolean(true);
// 方式3
var bool = Boolean(true);
```

**Array：**
```javascript
var arr = new Array();
var arr = new Array(size);
var arr = new Array(a1, a2, a3, a4, a5...);
```
拓展：[Array的方法和属性](https://www.w3school.com.cn/js/js_array_methods.asp)

**Date：**
```javascript
// 只能通过new方式初始化日期对象，但是可以传不同种类参数
var date = new Date()
var date = new Date(year, month, day, hours, minutes, seconds, milliseconds)
var date = new Date(milliseconds)
var date = new Date(date string)
```
拓展：[Date的方法和属性](https://www.w3school.com.cn/js/js_date_methods.asp)

**Math：**
```javascript
// 不用new，直接使用
var pi = Math.PI;
var sqrt = Math.sqrt(15);
```

**RegExp：**
```javascript
// 正则表达式
// 方式1 /匹配条件/模式修正符
/pattern/attributes
// 方式2
new RegExp(pattern, attributes);
```

拓展：[JavaScript HTML DOM对象](https://www.w3school.com.cn/js/js_htmldom.asp)

### JQuery基础知识

拓展：[JQuery选择器](https://www.runoob.com/jquery/jquery-ref-selectors.html)
拓展：[JQuery事件](https://www.runoob.com/jquery/jquery-ref-events.html)
拓展：[JQuery效果](https://www.runoob.com/jquery/jquery-hide-show.html)
拓展：[JQuery DOM操作](https://www.runoob.com/jquery/jquery-dom-get.html)

***
真题1：JavaScript中为id为test的元素设置样式(class)为good
```javascript
document.getElementById('test').className = 'good';
```
真题2：要求使用JQuery事件写在页面加载完成之后，动态绑定click事件到btnOK元素。
```javascript
$(function(){
	$('.btnOK').click(function(){
		// code...
	})
});
```

[**下一小节：2.2 AJAX基础**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/2-JavaScript%E3%80%81JQuery%E5%92%8CAJAX%E5%9F%BA%E7%A1%80/2-2AJAX%E5%9F%BA%E7%A1%80.md)

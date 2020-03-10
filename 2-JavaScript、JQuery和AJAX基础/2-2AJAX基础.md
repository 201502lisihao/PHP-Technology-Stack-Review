## 2.2 JavaScript、JQuery和AJAX基础 --  AJAX
***
### AJAX基本概念：

> Asynchronous JavaScript and XML （异步的JavaScript和XML）
> 
> 通过后台与服务器进行少量的数据交换，可以使网页实现异步更新

### AJAX基本工作原理：

> XMLHttpRequest是AJAX的基础，XMLHttpRequest是一个对象
> 
> XMLHttpRequest用于在后台与服务器交换数据

### XMLHttpRequest对象

**请求：**
```javascript
var xhttp = new XMLHttpRequest(); // 初始化XMLHttpRequest对象
xhttp.open(method, url, asycn); // 建立连接，method可选GET\POST
xhttp.send(string); // 发送请求，如果是GET请求不填参数
```

**响应：**
```javascript
onreadystatechange() // 当readyState属性发送变化时调用
readyState // 保存 XMLHttpRequest 的状态。0：请求未初始化、1：服务器连接已建立、2：请求已收到、3：正在处理请求、4：请求已完成且响应已就绪
responseText // 以字符串返回响应数据
responseXML // 以XML数据返回响应数据
status // 返回请求的状态号。200 OK、 403 Forbidden、 404 Not Found等
```

**示例：**
```javascript
<!DOCTYPE html>
<html>
<body>
	<h1>XMLHttpRequest 对象</h1>
	<p id="demo">让 AJAX 改变这段文本。</p>
	<button type="button" onclick="loadDoc()">更改内容</button>
	
	<script>
	function loadDoc() {
	  var xhttp;
	  if (window.XMLHttpRequest) {
	    // 针对现代浏览器的代码
	    xhttp = new XMLHttpRequest();
	  } else {
	    // 针对 IE6、IE5 的代码
	    xhttp = new ActiveXObject("Microsoft.XMLHTTP");
	  }
	  xhttp.onreadystatechange = function() {
	    if (this.readyState == 4 && this.status == 200) {
	      document.getElementById("demo").innerHTML = this.responseText;
	    }
	  };
	  xhttp.open("GET", "/example/js/ajax_info.txt", true);
	  xhttp.send();
	}
	</script>
</body>
</html>
```

### JQuery的AJAX操作

**常用方法：**

```javascript
$(selector).load(URL,data,callback); // 从服务器加载数据，它主要用于直接返回HTML的Ajax接口
$.get(URL,callback); // 通过 HTTP GET 请求向服务器请求数据
$.post(URL,data,callback); // 通过 HTTP POST 请求向服务器提交数据
$.ajax()
$.getJSON()
$.getScripte()
```

***
真题：AJAX技术利用了什么协议？简述AJAX的工作原理。

> AJAX主要利用了HTTP协议
> 
> Ajax其核心有 JavaScript、XMLHTTPRequest、DOM对象组成，通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据， 然后用JavaScript来操作DOM而更新页面。

[**下一小节：3.1 Linux基础**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/3-Linux%E5%9F%BA%E7%A1%80/3-1Linux%E5%9F%BA%E7%A1%80.md)
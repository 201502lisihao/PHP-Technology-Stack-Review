## 6.1 PHP框架基础 -- MVC原理和常见框架
***

此小节比较基础，一笔带过。

### MVC
> Model（数据操作层）、View（视图层）、Controller（业务处理层）
> 
> 一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码

### 单一入口工作原理
> index.php?r=user/login
> 
> r参数中，user是controller，login是对应方法
> 
> 实际原理就是框架根据路由实现(new UserController())->login()

**优点：**
> 统一进行安全检查
> 集中对程序进件处理

**缺点：**
> url不美观（绝大多数框架支持url重写，重写成www.xxx.com/user/login类似这种格式。）

### 模板引擎的理解
> PHP是一种html内嵌式的在服务端执行的脚本语言，但是PHP有很多可以使PHP代码和HTML代码分开的模板引擎，如：Smarty，Twig，Haml，Liquid等

**工作原理：**
> 实际是一个庞大的、完善的正则表达式匹配库


### 常见MVC框架
> Yii2、Laravel、ThinkPHP、Yaf

**Yaf：**
> Yaf使用PHP拓展的形式写的一个PHP框架，也就是以C语言为底层编写的，性能上要比纯PHP框架快一个数量级。

优点：
> 1. 执行效率高
> 2. 轻量级 
> 3. 可拓展性强

缺点：
> 1. 高版本兼容差，底层代码可读性差
> 2. 需要安装PHP拓展
> 3. 功能单一，开发需要编写大量插件

**Yii2：**
> Yii2是一款非常优秀的通用Web后端框架，结构简单优雅，实用功能丰富，拓展性强，性能高。

缺点：
> 1. 功能丰富，导致学习成本高
> 2. 框架量级较重


***
真题：Yii2框架如何实现数据的自动验证？

> Model中可以增加rule规则，用validator验证器根据rule规则对数据进行验证。


[**下一小节：7.1  常见算法**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/7-%E7%AE%97%E6%B3%95%E3%80%81%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/7-1%E5%B8%B8%E8%A7%81%E7%AE%97%E6%B3%95.md)
## 4.6 MySQL数据库 -- MySQL安全性
***
### SQL查询的安全方案

**1. 使用预处理语句防SQL注入**

看下面这种情况：

```sql
-- 假设我们的id是从GET方式的参数中获取
-- user/delete/?id=1
delete from user where id=1;

-- 如果有人恶意将GET参数改写：user/delete/?id=1 or 1=1
-- 我们的语句就变成了下面这样，所有的用户数据就都凉了
delete from user where id=1 or 1=1;
```

我们应当使用预处理语句来解决这样的风险：
```sql
-- 我们先用？占位，然后MySQL进行语法解析，使用prepare
delete from user where id=?;
-- 接下来向？绑定值，这样or这种关键字就不会参与解析了，也就失效了。
```

**2. 在应用层对写入数据库的数据进行特殊字符转义**

**3. 查询错误信息不用直接返回给用户，将错误记录到日志，然后返回统一格式的错误给用户**


### MySQL的其它安全设置

**1. 定期做数据备份**

**2. 不给查询用户root权限，合理分配权限**

**3. 关闭远程访问数据的权限**

**4. 修改root用户名称、修改root口令，不使用默认用户和口令**

**5. 删除多余用户**

**6. 限制一般用户浏览其他库**

**7. 限制用户对数据文件的访问权限**

***

真题1：SQL语句应该考虑哪些安全性？
> 请思考后写出答案

真题2：为什么使用PDO和MySQLi连接数据库比用原生的数据库函数连接要安全？
> 因为PDO和MySQLi支持预处理，可以有效预防SQL注入。原生数据库函数不支持预处理。

[**下一小节：5.1 程序功能设计**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/5-%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/5-1%E7%A8%8B%E5%BA%8F%E5%8A%9F%E8%83%BD%E8%AE%BE%E8%AE%A1.md)
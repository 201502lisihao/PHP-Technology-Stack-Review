## 4.1 MySQL数据库 -- MySQL基础
***

### 数据类型

**整数类型：**
```sql
TINYINT
SMALLINT
MEDIUMINT
INT
BIGINT
-- 都可以配置UNSIGNED属性，完成非负操作
-- MySQL中的 int(x)，x表示此列的显示宽度，x位；显示宽度经常会与zerofill一起使用，插入值不足x位，则左侧补零，超过x位，则正常显示。注意，显示宽度不会影响值得存储，仅仅影响外观
```

**实数类型：**
```sql
FLOAT
DOUBLE
-- FLOAT&DOUBLE类型支持使用标准的浮点进行近似计算，效率更高
DECIMAL
-- DECIMAL可以存储比BIGINT还大的整数；可以存储精确的小数（可以看成字符串存储大数）
```

**字符串类型：**
```sql
CHAR 
VARCHAR
-- CHAR是定长的，根据定义的长度分配足够的空间
-- CHAR会根据需要采用空格进行填充方便比较
-- CHAR适合存储很短的字符串或所有值都接近某一长度的字符串
-- CHAR超出设定的长度，会被截断
-- VARCHAR用于存储可变长字符，它比定长更节省空间
-- VARCHAR使用1或2个额外字节记录字符串长度，当长度小于255字节时使用一个，否则使用2个
-- VARCHAR如果存入数据超出指定长度，会被截断
-- 对于经常变更的数据，CHAR比VARCHAR更好，CHAR不容易产生碎片
-- 对于非常短的列，CHAR比VARCHAR效率更高
TEXT
BLOB
-- 尽量避免BLOB/TEXT类型，查询时会使用临时表，导致严重性能问题
```

**枚举类型：**
```sql
ENUM
-- 有时可以使用枚举代替常用的字符串类型
-- 把不重复的集合存储为一个预定义的集合
-- 值非常紧凑，把列表值压缩到1或者2个字节
-- 内部使用整数存储
-- 尽量避免使用数字作为ENUM枚举的常量，容易引起混淆
-- order by排序是按照内部存储的整数进行排序的
-- 枚举表会使表大小大大减小（因为内部使用整型存储）
```

**日期和时间类型：**
```sql
YEAR
-- yyyy格式的年份值
DATE
-- yyyy-mm-dd格式表示的日期值 ，以’HH:MM:SS’格式显示TIME值，但允许使用字符串或数字为TIME列分配值
TIME
-- hh:mm:ss格式表示的时间值，格式显示TIME值，但允许使用字符串或数字为TIME列分
TIMESTAMP
DATETIME
--yyyy-mm-dd hh:mm:ss格式，日期和时间的组合。格式显示DATETIME值，但允许使用字符串或数字为DATETIME列分配值

-- 尽量使用TIMESTAMP，比DATETIME效率高
-- 用整数保存时间戳的格式通常不方便处理
-- 如果需要存储微秒，可以使用BIGINT来存储
```

### MySQL基础操作

**连接与关闭：**
```
// -A 不预加载
mysql -u用户名 -p密码 -P端口 -h数据库地址 -D数据库名 --default-character-set=utf8 -A
```

**其它操作：**
```php
// \G 格式化输出,垂直显示
select * from test\G
// \c 取消当前操作
// \q 退出mysql
// \s 显示当前的服务器状态
// \h 显示帮助信息
// \d 改变执行服务的设置（比如把结尾分号改掉）

```

### MySQL数据表引擎

**InnoDB：**

> 1. 默认事务型引擎，最重要最广泛的存储引擎，性能非常优秀
> 2. 数据存储在共享表空间，可以通过mysql配置文件分开
> 3. 对主键查询性能高于其他类型
> 4. 内部做了很多优化，从磁盘读取数据时自动在内存建立hash索引，插入数据时自动构建插入缓冲区
> 5. 通过一些机制和工具支持真正的热备份
> 6. 支持崩溃后的安全恢复
> 7. 支持行级锁
> 8. 支持外键

**MyISAM：**
> 1. 5.1版本前，MyISAM是默认存储引擎
> 2. 拥有全文索引、压缩、空间函数
> 3. 不支持事务、行级锁、不支持崩溃后的安全恢复
> 4. 表存储在2个文件，MYD（存数据）和MYI（存索引）
> 5. 设计简单，某些场景下性能很好

**其它表引擎：**
```sql
Archive
Blackhole
CSV
Memory
...
```

`tip：`优先使用InnoDB，性能优先



### MySQL锁机制

> 锁是日常开发中常见的问题，当多个查询同一时刻对数据进行修改时，就会产生并发控制的问题。
> 
> 分为`共享锁（读锁）`和`排它锁（写锁）`

共享锁（读锁）：

> 共享的，不阻塞，多个用户可以读同一个资源，互不干扰

排它锁（写锁）：

> 排它的，一个写锁会阻塞其它读锁和写锁，这样只允许一个人进行写入，防止其它用户读取正在写入的资源。

锁粒度：

> MyISAM使用表级锁，系统开销小。
> 
> InnoDB使用行级锁，最大程度支持并发，但也带来了最大的锁开销。

### MySQL事务处理

> 1. MySQL提供事务处理的表引擎是InnoDB。
> 2. 服务器不管理事务，由下层引擎实现，所以同一事务中，使用多种存储引擎是不可取的。比如一个事务中涉及到的2张表不全是InnoDB。
> 3. 在非事务的表上执行事务操作，MySQL不会发出提示，也不会报错。

### 存储过程

> 1. 为以后的使用而保存的一条或多条MySQL的语句集。
> 2. 存储过程就是业务逻辑和流程的集合，类似于SQL下的自定义程序。
> 3. 可以在存储过程中创建表、更新数据、删除等操作。

```sql
mysql> delimiter $$　　-- 将语句的结束符号从分号;临时改为两个$$(可以是自定义)
mysql> CREATE PROCEDURE delete_matches(IN p_playerno INTEGER)
    -> BEGIN
    -> 　　DELETE FROM MATCHES
    ->    WHERE playerno = p_playerno;
    -> END$$
Query OK, 0 rows affected (0.01 sec)
 
mysql> delimiter;　　-- 将语句的结束符号恢复为分号
```

### MySQL触发器

**使用场景：**

> 1. 可通过数据库中的一张表实现级联修改。
> 2. 实时监控某张表的某个字段的变更，从而做出对应处理。
> 3. 某些业务编号的处理

```sql
mysql> drop trigger if exists databaseName.tri_Name;  
mysql> create trigger tri_Name  -- tri_Name代表触发器名称
tirgger_time trigger_event on tableName  -- tirgger_time为触发时机，可选值有after/before，trigger_event为触发事件，可选值有insert/update/delete
for each row   -- 这句话在mysql是固定的，表示任何一条记录上的操作满足触发事件都会触发该触发器。 
begin  
    sql语句;  
end
```

`tip：`滥用会造成数据库及应用程序的维护困难

***
真题1：请写出下面MySQL数据类型表达的意义（int(0)、char(16)、varchar(16)、datetime、text）。

> 请认真思考后写下答案。

真题2：说明表存储引擎InnoDB和MyISAM的区别。

> 1. InnoDB支持事务，MyISAM不支持。
> 2. InnoDB数据存在共享表空间，MyISAM数据存在MYD文件，索引存在MYI文件。
> 3. InnoDB支持行级锁，MyISAM支持表级锁。
> 4. InnoDB支持崩溃后的安全修复，MyISAM不支持。

[**下一小节：4.2 创建高性能索引**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/4-MySQL%E6%95%B0%E6%8D%AE%E5%BA%93/4-2MySQL%E5%88%9B%E5%BB%BA%E9%AB%98%E6%80%A7%E8%83%BD%E7%B4%A2%E5%BC%95.md)
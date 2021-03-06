## 4.5 MySQL数据库 -- MySQL高可用&高可拓展
***
### 分区表

**原理**
> 对用户而言，分区表是一个独立的逻辑表，但是在MySQL底层是将其分成了若干个物理子表来存储，这对用户是透明的，但是每一个分区表都是一个独立的表文件。
> 
> 创建表时使用partition by字句定义每一个分区存放的数据，执行查询时，优化器会根据分区定义过滤哪些没有我们需要数据的分区，这样查询范围将大大缩小。

```sql
-- 创建一个分区表，[id<10]，[10 <= id < 20]， [id >= 20]，符合条件的数据将会落在对应的fenqub
create table user (
	id int,
	user_name varchar(20)
) engine innodb
partition by range(id) (
partition p1 values less than (10),
partition p2 values less than (20),
partition p3 values less than MAXVALUE
);
```

**适用场景**
> 1. 表非常大，无法全部存在内存，或者只使用最后加入的热点数据，其它都是历史数据的表。
> 2. 分区表数据更易维护，可以对独立的分区进行独立的操作。
> 3. 分区表的数据可以分布在不同机器上，从而高效的利用机器资源。

当然，分区表也有一些限制，这个偏dba，不做赘述。

### 分库分表

**原理**
> 通过hash算法或工具实现将一张数据表`水平`或`垂直`进行物理切分。

**适用场景**
> 1. 数据表条数到达百万、千万或更大时。
> 2. 解决表锁问题时。

**水平分表**

使用场景：
> 1. 各条数据之间本身就具有独立性。
> 2. 数据条数过大。
> 3. 需要把数据放在不同介质上（如热点数据缓存、热点数据使用高性能服务器等）。

缺点：
> 1. 给应用层增加复杂度，查询时需要计算表名，查询所有的数据都需要UNION操作。
> 2. 在许多情况下，这种复杂度会超过分表带来的优点，降低效率。

**垂直分表**

使用场景：
> 1. 一些列常用，一些列不常用。
> 2. 可以使数据行变小，一个数据页能存储更多地数据，查询时减少I/O次数。

缺点：
> 1. 查询冗余列时，都需要进行JOIN操作。


**分库分表缺点：**
> 1. 有些分表策略基于应用层的分表算法，一旦逻辑算法改变，整个分表逻辑就会改变，拓展性很差。
> 2. 对于应用层来说，会增加开发成本。

### MySQL复制原理及负载均衡

**主从复制原理：**
> 1. 主库将数据变更记录到二进制日志中（binlog）。
> 2. 从库将主库的binlog复制到自己的中继日志。
> 3. 从库读取中继日志中的事件，将其重放到从库中，完成数据同步。

**主从复制解决了哪些问题？**
> 1. 数据分布：随意停止和开始复制，并在不同地理位置分布数据备份。
> 2. 负载均衡：降低单个服务器压力。
> 3. 高可用&故障切换：帮助应用程序避免单点失败。
> 4. 升级测试：高版本MySQL升级时，可以先升级个别从库做测试。

***
真题1：简述MySQL分区操作和分表操作原理，分别说是分区和分表的使用场景和优缺点。
> 思考后写出自己的答案

真题2：设定网站的用户数量在千万级，但是活跃用户的数量只有1%，如果通过优化数据库来提供活跃用户的访问速度？

> 1. 可以使用MySQL分区，将活跃用户和不活跃用户分区，这样查询活跃用户时，MySQL将在小范围的分区内进行查询，大大提升速度。
> 2. 也可以在业务上对用户水平分表，将活跃用户和不活跃用户区分开。

[**下一小节：4-6 MySQL安全性**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/4-MySQL%E6%95%B0%E6%8D%AE%E5%BA%93/4-6MySQL%E5%AE%89%E5%85%A8%E6%80%A7.md) 

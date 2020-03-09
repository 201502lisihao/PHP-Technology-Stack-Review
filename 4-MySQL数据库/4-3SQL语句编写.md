## 4.3 MySQL数据库 -- SQL语句编写
***

增删改查比较基础，这里不做赘述，需要单独复习的同学[点击这里](https://www.runoob.com/mysql/mysql-insert-query.html)。

我们直接来复习关联查询。

### MySQL的关联查询语句

**六种关联查询：**
```sql
1. CROSS JOIN（交叉连接）
2. INNER JOIN（内连接）
3. LEFT JOIN/RIGHT JOIN（外连接）
4. UNION/UNION ALL（联合查询）
5. FULL JOIN（全连接）
6. 嵌套查询
```

交叉连接：
```sql
-- 没有任何关联条件，结果是笛卡尔积（直积），结果集会很大，没有意义，很少使用
SELECT * FROM  A,B(,C);
或
SELECT * FROM A CROSS JOIN B (CROSS JOIN C);
```

内连接：
```sql
-- 多表汇总同时符合某种条件的数据记录的集合，以条件为主，满足连接条件则显示
SELECT * FROM A,B WHERE A.id = B.user_id;
或
SELECT * FROM A INNER JOIN B ON A.id = B.uid;
或
SELECT * FROM A JOIN B ON A.id = B.uid;

-- 内连接分为三类
-- 等值连接：ON A.id = B.id
-- 不等值连接：ON A.id > B.id
-- 自连接：SELECT * FROM A T1 INNER JOIN A T2 ON T1.id = T2.pid;
```

外连接：
```sql
-- 左外连接：
-- LEFT OUTER JOIN，以左表为主，按照ON后的关联条件匹配右表，没有匹配到则用NULL填充，可以简写成LEFT JOIN
SELECT * FROM A LEFT JOIN B ON A.id = B.uid;

-- 右外连接：
-- RIGHT OUTER JOIN，以右表为主，按照ON后的关联条件匹配左表，没有匹配到则用NULL填充，可以简写成RIGHT JOIN
SELECT * FROM A RIGHT JOIN B ON A.id = B.uid;
```

联合查询：
```sql
-- 就是把多个结果集几种在一起，以UNION前的结果为基准
-- 需要注意的是联合查询的列数要相等，相同的记录行会合并
-- UNION ALL和UNION的区别是，结果集不会合并重复的记录行
SELECT * FROM A_1 UNION SELECT * FROM A_2;
```

全连接：
```sql
-- MySQL不支持全连接
-- 可以使用LEFT JOIN和RIGHT JOIN联合使用实现全连接
SELECT * FROM A LEFT ON B ON A.id = B.user_id UNION SELECT * FROM A RIGHT JOIN B ON A.id = B.user_id;
```

嵌套查询：
```sql
-- 用一条SQL语句的结果作为另外一条SQL语句的条件
SELECT * FROM A WHERE id IN (SELECT ID FROM B);
```

***
真题1：有A(id,sex,par,c1,c2)，B(id,age,c1,c2)两张表，其中A.id和B.id关联，现要求写出一条SQL语句，将B中age>50的记录的c1，c2更新到A表的c1，c2字段中。

```sql
-- 方式1
UPDATE A,B SET A.c1 = B.c1, A.c2 = B.c2 WHERE A.id = B.id AND B.age > 50;

-- 方式2
UPDATE A INNER JOIN B on A.id = B.id SET A.c1 = B.c1, A.c2 = B.c2 WHERE B.age > 50;
```

真题2：为了记录足球比赛的结果，设计表如下：

team表：
| 字段名 | 类型 | 描述 |
| :--------: | :--------: | :------: |
| teamId | int | 主键 |
| teamName | varchar(20) | 队伍名称 |

matchs表：
| 字段名 | 类型 | 描述 |
| :--------: | :--------: | :------: |
| matchId | int | 主键 |
| hostTeamId | int | 主队Id |
| guestTeamId | int | 客队Id |
| matchResult | varchar(20) | 比赛结果 |
| matchDate | date | 比赛日期 |

其中match赛程表中的hostTeamId与guestTeamId和team表中的teamId关联。查询2006-06-01到2006-07-01之间举行的所有比赛，并用以下形式列出：

> 拜仁 2:0 不莱梅 2006-06-21

```sql
SELECT t1.teamName, m.matchResult, t2.teamName, m.matchDate FROM matchs AS m LEFT JOIN team AS t1 ON m.hostTeamId = t1.teamId, LEFT JOIN team AS t2 ON m.guestTeamId = t2.teamId WHERE m.matchData between "2006-06-01" and "2006-07-01";
```

[**下一小节：4.4 MySQL查询优化**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/4-MySQL%E6%95%B0%E6%8D%AE%E5%BA%93/4-4MySQL%E6%9F%A5%E8%AF%A2%E4%BC%98%E5%8C%96.md)
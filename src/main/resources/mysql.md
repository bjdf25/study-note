> count(1) == count(*)
>
> DDL:create drop alter
>
> DML:insert delete
>
> DQL:select shere
>
> DCL:grant

写法顺序：select--->from --->where --->group by --->having --->order by

执行顺序：from---> where --->group by --->having---> select---> order by

**where字段后面不能跟聚合函数！**

**要使用聚合函数就要配合group by使用，having字段必须和group by语句一起使用，group by的作用是限定分组条件，而having则是对group by中分出来的组进行条件筛选。**

group by易错知识点：

1. 使用group by子句中，**select子句不能出现聚合键之外的列名**。总结就是select子句中只能存在下面三种元素：
    - 常数
    - 汇总函数（聚合函数）
    - group by子句中指定的列名（聚合键）
2. group by子句不能使用select子句中定义的别名， 是因为sql先执行的group by，再执行select。
3. group by子句的结果是无序的。

- **having子句和select语句一样只能存在上面三种元素**
- 聚合键所对应的条件建议写在where子句中，而不是having子句中，可以提高运算速度，他们两者的作用区别可以理解为：
    - where子句=指定行所对应的条件
    - having子句=指定组所对应的条件

注意：在order by子句中可以使用聚合函数或者select子句中未使用的列

> case when
>
> then 1
>
> else 2

候选索引与主键索引一样需要字段的唯一性。

### 子查询：

> 子查询就是将用来定义视图的select语句直接用于from子句当中，还可以放在where子句中与in，all，any配合使用，简单一点说就是把一个查询的结果在另一个查询中使用就叫子查询。

- **子查询不会想视图那样保存在硬盘中，而是在select语句执行之后就消失了，因此子查询就是一张一次性的视图。**
- **在sql运行顺序中会先执行子查询，然后再执行外部语句。**
- **where x in （select x）**

子查询注意事项：

- 避免使用多层嵌套子查询，不容易看懂，也容易弄错。
- 子查询最好重新命名子查询名称（子查询 AS 子查询名称） 便于理解该子查询的用途和目的，类似于编程中变量命名。

#### 标量子查询：

> 标量子查询是指子查询返回的是单一值（一个数字或一个字符串）的标量，也是子查询中最简单的返回形式。

选取大于平均成绩学生的学号和成绩，错误写法：

```sql
select 学号，成绩 from score where 成绩 > AVG（成绩）;
```

**这样的写法程序会报错，因为where子句后面不允许使用聚合函数**，正确写法：

```sql
select 学号，成绩 from score where 成绩 > (select AVG(成绩) from score);
```

#### 关联子查询：

> 对于外部查询返回的每一行数据，内部查询都要执行一次，在关联子查询中信息流是双向的。外部查询的每行数据传递一个值给子查询，然后子查询为每一行数据执行一次并返回它的纪录。然后外部查询根据子查询返回的记录做出决策。

**在每个组里面比较一般会用到关联子查询：**查找出每个课程成绩都大于该课程平均成绩的学生。

```sql
select 学号，成绩，课程号 from score as s1 --同一个表要使用别名，便于区分 where 成绩 > (select AVG(成绩) from score as s2 --同一个表要使用别名，便于区分 where s1.课程号 = s2.课程号 group by 课程号 ) 
```

注意：关联条件一定要写在子查询中，且s2仅在子查询里面有效！

### 多表查询：

##### 设计多表时要考虑表与表之间的关系：

- 一对一

    - 外键

    - 共同主键

- 一对多

    - 在多的表中建立一的表的主键所对应的外键。

- 多对多

    - 建立中间表存储两个多表主键所对应的外键。

##### union：将两个表的数据按照行合并在一起，并且重复值只保留一个

```sql
select 课程号，课程名称 from course union select 课程号，课程名称 from course1
```

如果想保留重复值，那么就使用union all，*需要注意的是，这里的重复是指两张表中课程号和课程名称完全相同的行，其他列是否重复不影响查询结果。即重复值只针对select子句中查询的字段*。

##### 内联结(inner join)：查找出同时存在于两张表的数据。两张表中满足于关联条件的所有数据都查找出来。

```sql
select a.学号,a.姓名,b.课程号 from student as a inner join score as b on a.学号 = b.学号;
```

![内联结](images\内联结.png)

#### 左联结(left join)：

> 取出左边表的全部数据，右边的表选出与左边相同数据的行，然后进行数据合并。

```sql
select a.学号，a.姓名,b.课程号 from student as a left join score as b on a.学号 = b.学号;
```

![左联结示意图](images\左联结示意图.png)

**如果想查找在左表中有而在右表中没有的数据**：只需限定右表关联条件为空即可。

![左表相对右表没有的](images\左表相对右表没有的.png)

```sql
select a.学号，a.姓名,b.课程号 from student as a left join score as b on a.学号 = b.学号 where b.学号 = null;
```

**当有多个外连接的时候，会按照从左向右的顺序执行sql语句，即先用A表连接B表，生成一张临时中间表再和C表进行连接。**

#### 全联结(full join):

> 取出左表和右表的所有数据，有相同数据就合并，没有则用null来填充。

**mqsql不支持全联结。**

### case：条件函数

![case应用](images\case应用.png)

![case应用2](images\case应用2.png)

### **通过explain语句可以分析，mysql是如何执行这条sql语句的。**

### 索引：

- 主键索引：非空且唯一
- 唯一索引：非空
- 全文索引
- 普通索引

1. 一般来说创建表的时候都会指定主键索引，如果创建表时没有指定主键索引，也可以在创建表后再添加：

```sql
alter table user add primary key (列名);
```

2. 一般来说，普通索引的创建，是先创建表，然后再创建普通索引：

```sql
create index 索引名 on 表 (列1，列2);
```

3. 全文索引，不经常用，全文搜索常用elasticsearch实现。
4. 唯一索引，unique字段可以为null，但是如果是具体内容，则不能重复

```sql
create unique index 索引名 on 表 （列1，列2);
```

5. 删除索引：

```sql
alter table 表名 drop index 索引名;
```

#### SQL语句的小技巧：

1. 在使用group by分组查询时，分组后会默认排序，可能会降低速度，在group by后面增加order by null就可以防止排序。

2. 有些情况下可以用连接来代替子查询。因为使用join，mysql不需要再内存中创建临时表。

   ```sql
   select * from dept, emp where dept.deptno = emp.deptno;[简单处理]
   select * from dept left join emp on dept.deptno = emp.deptno;[连接更ok]
   ```



**<selectKey>：insert的时候当插入表id为自增时需要用到selectkey查找last_insert_id**

**MySQL的唯一联合索引中若是有一个或以上字段为null时，该索引的唯一性将失效**

关于insert ignore into:忽略仅仅是忽略主键或者索引重复的数据，如果主键或者索引没有重复，即使其他数据重复了也依然能够插入数据。**问题出在如果主键是自增的话，插入数据的时候主键已经自增一了，则主键永远不可能重复，就永远达不到忽略重复数据的效果。解决办法：把想要保证不重复的字段也加上索引**。  发生场景：表中仅有id为主键，当其他字段插入重复数据的时候也能插入成功。除非把想要不重复的字段加上联合唯一索引。

特别说明：在MYSQL中UNIQUE索引将会对null字段失效，也就是说(a字段上建立唯一索引)：

```
 INSERT INTO `test` (`a`) VALUES (NULL);1
```

是可以重复插入的（联合唯一索引也一样）。

#### can't update when select:

**无法在查询一张表的时候对一张表进行更改。如果想要达到效果可以把查出来的结果封装成一个中间表：**

```sql
delete from schema.table where t.parent_id in (select temp.id from (select id from schema.table where sex = 'man') temp);
```

**1.INSERT INTO SELECT****语句**

语句形式为：

```sql
Inser into Table2(field1,field2,…) select value1,value2,… from Table1
或者：
Insert into Table2 select *  from Table1
```

注意：（1）要求目标表Table2必须存在，并且字段field,field2…也必须存在

（2）注意Table2的主键约束，如果Table2有主键而且不为空，则 field1， field2…中必须包括主键

（3）注意语法，不要加values，和插入一条数据的sql混了，不要写成:

Insert into Table2(field1,field2,…) values (select value1,value2,… from Table1)

由于目标表Table2已经存在，所以我们除了插入源表Table1的字段外，还可以插入常量

**2.SELECT INTO FROM****语句**

语句形式为：

```sql  
SELECT vale1, value2 into Table2 from Table1
```

要求目标表Table2不存在，因为在插入时会自动创建表Table2，并将Table1中指定字段数据复制到Table2中。


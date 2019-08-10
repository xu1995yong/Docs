## SQL基础关键字

###  DISTINCT 关键字

有时候，数据表中可能会有重复的记录。**DISTINCT** 关键字同 SELECT 语句一起使用，可以去除所有重复记录，只返回唯一项。

**DISTINCT** 关键字也可以同SUM、COUNT等分组函数一起使用，先去除重复值后再使用分组函数计算。



### SQL中的函数

#### COUNT

1. COUNT(column_name) 函数返回指定列的值的数目（**NULL 不计入**）：

```sql
SELECT COUNT(column_name) FROM table_name
```

2. COUNT(*) 函数返回表中的记录数：

```sql
SELECT COUNT(*) FROM table_name
```



### GROUP BY 子句

GROUP BY 子句用于根据 BY 指定的规则对数据进行分组，所谓的分组就是将一个“数据集”划分成若干个“小区域”，然后针对若干个“小区域”进行数据处理。

GROUP BY 子句必须在 WHERE 子句的条件之后，ORDER BY 子句之前。

GROUP BY 子句可以包含多个分组条件，使用逗号分隔。

**Group By中Select指定的字段限制：**在select指定的字段要么就要包含在Group By语句的后面，作为分组的依据；要么就要被包含在聚合函数中。



分组筛选 HAVING子句 分组后筛选



#### SQL中的连接查询

### 基本语法

```sql
SELECT
	查询列表 
FROM
	表 1
	连接类型 JOIN 表 2 ON 连接条件 
	连接类型 JOIN 表 3 ON 连接条件 
WHERE
	筛选条件
```

### 查询语句的逻辑执行顺序

```sql
sql执行顺序 
(1) from 
(3) join 
(2) on 
(4) where 
(5) group by(开始使用select中的别名，后面的语句中都可以使用)
(6) AVG(),SUM()···
(7) having 
(8) select 
(9) distinct 

(10) order by
(11) limit 
```

### 内连接





#### 等值连接

#### 非等值连接

#### 自连接



### 外连接

左外连接

右外连接

全外连接



## 子查询





#### 常见题目

1. 查询没学过“叶平”老师课的同学的学号、姓名；

```sql
# 1. 李老师教过的所有课程的课程号
SELECT cid FROM course INNER JOIN teacher on course.tid = teacher.tid  WHERE teacher.tname like '李%'
# 2. 选了李老师教过的所有课程的学生id
SELECT sid FROM sc INNER JOIN course on sc.cid = course.cid INNER JOIN teacher on course.tid = teacher.tid  WHERE teacher.tname like '李%'
# 3. 查询没学过“叶平”老师课的同学的学号、姓名；
select student.* from student where sid not in(
SELECT sid FROM sc INNER JOIN course on sc.cid = course.cid INNER JOIN teacher on course.tid = teacher.tid  WHERE teacher.tname like '李%')
```

2. 查询既学过“001”并且也学过编号“002”课程的同学的学号、姓名；

```sql
法一：
	SELECT
student.* 
	FROM
	student
INNER JOIN sc ON student.sid = sc.sid 
	WHERE
	sc.cid = 1 
AND sc.sid IN ( SELECT sid FROM sc WHERE sc.cid = 2 )
#方法二：
	SELECT
s1.* 
	FROM
	( SELECT student.* FROM student INNER JOIN sc ON student.sid = sc.sid WHERE sc.cid = 1 ) as s1
INNER JOIN sc ON sc.sid = s1.sid 
	WHERE
	sc.cid = 2
```


3. 查询学过“叶平”老师所教的所有课的同学的学号、姓名；

```sql
#方法一
SELECT
   	sid 
   FROM
   	sc
   	INNER JOIN course ON sc.cid = course.cid
   	JOIN teacher ON course.tid = teacher.tid 
   WHERE
   	teacher.tname = '张三' 
    GROUP BY
   	sid 
   HAVING
   	count( * ) = ( SELECT count( * ) FROM course JOIN teacher ON course.tid = teacher.tid WHERE teacher.tname = '张三' )
   	
#方法二：
```

4. 查询所有课程成绩小于60分的同学的学号、姓名；
```sql
SELECT
	DISTINCT(sid) 
FROM
	sc
WHERE
	sid NOT IN ( SELECT sid FROM sc GROUP BY sc.sid HAVING max( sc.score ) >= 60 )
```

5. 查询没有学全所有课的同学的学号、姓名；

   ```sql
   SELECT
   	sid 
   FROM
   	sc 
   GROUP BY
   	sid 
   HAVING
   	count( * ) < ( SELECT count( * ) FROM course )
   ```

6. 查询和“1002”号的同学学习的课程完全相同的其他同学学号和姓名； 

   ```sql
   SELECT
   	sid 
   FROM
   	sc 
   WHERE
   	sc.cid IN ( SELECT cid FROM sc WHERE sc.sid = 1 ) 
   GROUP BY
   	sid 
   HAVING
   	count( * ) = ( SELECT count( * ) FROM sc WHERE sc.sid = 1 )
   ```

   

## Mysql中的Limit子句

### 1. Limit的语法

**LIMIT 子句用于强制 SELECT 语句返回指定行数的记录。**

LIMIT 子句接受一个或两个数字参数。

- 如果只给定一个参数，它表示返回的记录行数目。
- 如果给定两个参数，则第一个参数指定从第几个记录行开始，第二个参数指定返回记录行的数目。初始记录行的偏移量是0。为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1。

```sql
SELECT * FROM table where age >20 LIMIT 5,10; # where子句查询出数据后，从第5个记录行开始，返回10个记录行
```

### 2. Limit中offset过大时的优化



1. 准备测试数据表及数据

  ```sql
    CREATE TABLE `member` (
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    `name` varchar(10) NOT NULL COMMENT '姓名',
    `gender` tinyint(3) unsigned NOT NULL COMMENT '性别',
    PRIMARY KEY (`id`),
    KEY `gender` (`gender`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

  ```

2. 出现的问题机原因分析
    当offset很大时，会出现效率问题，随着offset的增大，执行效率下降。 

  ```sql
select * from member where gender=1 limit 300000,1;
  ```

2. 出现的问题机原因分析
   当offset很大时，会出现效率问题，随着offset的增大，执行效率下降。 

```sql
select * from member where gender=1 limit 300000,1;
```


2. 出现的问题机原因分析
    当offset很大时，会出现效率问题，随着offset的增大，执行效率下降。 

  ```sql
select * from member where gender=1 limit 300000,1;
  ```

根据InnoDB索引的结构，查询过程为：

- 通过二级索引查到主键值（找出所有gender=1的id)。
- 再根据查到的主键值通过主键索引找到相应的数据块（根据id找出对应的数据块内容）。
- 根据offset的值，查询300001次主键索引的数据，最后将之前的300000条丢弃，取出最后1条。

可以看出，问题出现在通过主键索引再去主键索引表中多次IO去找相应的数据块。

因此可以证实，mysql查询时，offset过大影响性能的原因是多次通过主键索引访问数据块的I/O操作。

3. 优化

如果在找到主键索引后，先执行offset偏移处理，跳过300000条，再通过第300001条记录的主键索引去读取数据块，这样就能提高效率了。


**因此我们先查出偏移后的主键，再根据主键索引查询数据块的所有内容即可优化。**

```sql
select a.* from member as a inner join (select id from member where gender=1 limit 300000,1) as b on a.id=b.id;
```
# mysql进阶

## 4. InnoDB 事务

1. 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
2. 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
3. 事务用来管理 insert,update,delete 语句

### 4.1 事务是必须满足4个条件（ACID）

​	**原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
​	**一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
​	**隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
​	**持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### 4.2 事务控制语句：

- BEGIN 或 START TRANSACTION 显式地开启一个事务；
- COMMIT 也可以使用 COMMIT WORK，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；
- ROLLBACK 也可以使用 ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；
- SAVEPOINT identifier，SAVEPOINT 允许在事务中创建一个保存点，一个事务中可以有多个 SAVEPOINT；
- RELEASE SAVEPOINT identifier 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
- ROLLBACK TO identifier 把事务回滚到标记点；
- SET TRANSACTION 用来设置事务的隔离级别。InnoDB 存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE。

**MYSQL 事务处理主要有两种方法：**

1、用 BEGIN, ROLLBACK, COMMIT来实现（显式事务）

- **BEGIN** 开始一个事务
- **ROLLBACK** 事务回滚
- **COMMIT** 事务确认

2、直接用 SET 来改变 MySQL 的自动提交模式:（隐式事务）

innodb中每一个DML操作都是隐式事务，默认隐式事务自动提交。

- **SET AUTOCOMMIT=0** 禁止自动提交
- **SET AUTOCOMMIT=1** 开启自动提交

### 4.3 数据库隔离级别

| 事务的隔离级别   | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| Read uncommitted | √    | √          | √    |
| Read committed   | ×    | √          | √    |
| Repeatable read  | ×    | ×          | √    |
| Serializable     | ×    | ×          | ×    |





读未提交（Read uncommitted）

读提交（read committed）

可重复读（repeatable read）

串行化（Serializable

丢失修改 脏读 幻读 不可重复度





​	
​	



------

索引：（未完）

show index from tblname;	

索引是对数据库表中一或多个列的值进行排序的结构，是帮助MySQL高效获取数据的数据结构。
（数据库是磁盘文件，磁盘IO 的代价较高，所以采用索引减少IO 次数）

索引根据不用方式分类。

按索引相关列个数：单列索引、组合索引。

按索引约束：普通索引、唯一索引、主键索引。

按索引基本原理：B+ 树索引，哈希索引，全文索引，R-TREE 索引（空间索引，主要用于地理空间数据类型，很少使用）。

按索引与数据的关系：聚集索引、非聚集索引。

http://blog.csdn.net/qq_33556185/article/details/52192551
https://www.cnblogs.com/hyd1213126/p/5828937.html

## 普通索引

### 创建索引

这是最基本的索引，它没有任何限制。它有以下几种创建方式：

```
CREATE INDEX indexName ON mytable(username(length)); 
```

如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。

### 修改表结构(添加索引)

```
ALTER table tableName ADD INDEX indexName(columnName)
```

### 创建表的时候直接指定

```
CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
INDEX [indexName] (username(length))  
 
);  
```

### 删除索引的语法

```
DROP INDEX [indexName] ON mytable; 
```









普通索引

​    普通索引(由关键字KEY或INDEX定义的索引)的唯一任务是加快对数据的访问速度。因此，应该只为那些最经常出现在查询条件(WHERE column = ...)或排序条件(ORDER BY column)中的数据列创建索引。只要有可能，就应该选择一个数据最整齐、最紧凑的数据列(如一个整数类型的数据列)来创建索引。 
​	还可以将两个列创建为组合索引。

​	创建普通索引三种方法：

​	其中length为截取长度。如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。数值类型不可以加length。
​	index_name为索引名，后两种方式可省略，第一种不可以。

```mysql
CREATE INDEX index_name ON table_name (column_name(length),column_name(length)); 
ALTER table table_name ADD INDEX index_name(column_name)
CREATE TABLE mytable(  
	id INT NOT NULL,   
	username VARCHAR(16) NOT NULL,  
	INDEX index_name (username(3))
); 
```



​		1.CREATE INDEX index_name ON table_name (column_name(length),column_name(length)); 
​		
​		2.ALTER table tableName ADD INDEX indexName(columnName)
​		3.CREATE TABLE mytable(  
​		id INT NOT NULL,   
​		username VARCHAR(16) NOT NULL,  
​		INDEX [indexName] (username(length))  
​		); 

删除索引

```mysql
DROP INDEX index_name ON table_name;//删除索引
SHOW INDEX FROM table_name;//没起名字的索引，系统会帮起，可用此条查看
```





唯一索引

​	

  主键和唯一索引都要求值唯一，但是它们还是有区别的：

1. 主键是一种约束，唯一索引是一种索引；
2. 一张表只能有一个主键，但可以创建多个唯一索引； 
3. 主键创建后一定包含一个唯一索引，唯一索引并不一定是主键；
4. 当没有设定主键时，非空唯一索引自动称为主键。
5. 主键不能为null，唯一索引可以为null；
6. 主键可以做为外键，唯一索引不行；  

主键索引



















### 索引的优缺点？那些情况适合建索引那些情况不适合建索引？

索引的优点：

通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。
可以大大加快 数据的检索速度，这也是创建索引的最主要的原因。
可以加速表和表之间的连接，特别是在实现数据的参考完整性方面特别有意义。
在使用分组和排序 子句进行数据检索时，同样可以显著减少查询中分组和排序的时间。
通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。
索引的缺点

创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加。
索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。
当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。
哪些情况需要加索引？

在经常需要搜索的列上，可以加快搜索的速度；
在作为主键的列上，强制该列的唯一性和组织表中数据的排列结构；
在经常用在连接的列上，这些列主要是一些外键，可以加快连接的速度；
在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的；
在经常需要排序的列上创建索引，因为索引已经排序，这样查询可以利用索引的排序，加快排序查询时间；
在经常使用在WHERE子句中的列上面创建索引，加快条件的判断速度。
哪些情况不需要加索引？

对于那些在查询中很少使用或者参考的列不应该创建索引。这是因为，既然这些列很少使用到，因此有索引或者无索引，并不能提高查询速度。相反，由于增加了索引，反而降低了系统的维护速度和增大了空间需求。
对于那些只有很少数据值的列也不应该增加索引。这是因为，由于这些列的取值很少，例如人事表的性别列，在查询的结果中，结果集的数据行占了表中数据行的很大比例，即需要在表中搜索的数据行的比例很大。增加索引，并不能明显加快检索速度。
对于那些定义为text, image和bit数据类型的列不应该增加索引。这是因为，这些列的数据量要么相当大，要么取值很少。
当修改性能远远大于检索性能时，不应该创建索引。这是因为，修改性能和检索性能是互相矛盾的。当增加索引时，会提高检索性能，但是会降低修改性能。当减少索引时，会提高修改性能，降低检索性能。因此，当修改性能远远大于检索性能时，不应该创建索引。







## mysql存储引擎不同点

1.MyISAM和InnoDB两个存储引擎的索引实现方式不同

https://blog.csdn.net/mine_song/article/details/63251546

****************

mysql性能调优：（未完）
1.调整好数据类型
2.常搜索的列加index
3.选好引擎？
4.行级锁表记锁事务？
5.优化sql？
		比如模糊搜索的%要用，就要放后面搜索
		in exists效率区别



。。。百度一下

存储引擎使用MyISAM是因为此引擎没有事务，插入速度极快，方便我们快速插入千万条测试数据，等我们插完数据，再把存储类型修改为InnoDB。

./其他文章

************************

create procedure（未完）
 必须调delimiter //







**********************

## 存储引擎区别

​	mysql的最大优点就是插件式存储引擎，一个表对应一个存储引擎，特殊情况用特殊的存储引擎，甚至可以自定义存储引擎。

​	
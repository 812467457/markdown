# MySQL高级

## 一、索引优化

### 1、SQL性能下降的原因

* SQL语句的问题

* 索引失效：如查询最多的字段没有索引，select * from user where name = "";
  * 单值索引：`create index idx_user_name on user(naem);`
  * 复合索引：`create index idx_user_nameEmali on user(naem,email);`
* 关联查询的join太多
* 服务器问题

### 2、7种Join

* 左外连接：`select * from table_a a left join table_b b on a.key = b.key `
* 内连接：`select * from table_a a inner join table_b b on a.key = b.key`
* 右外连接：`select * from table_a a right join table_b b on a.key = b.key`
* 左连接：`select * from table_a a left join table_b b on a.key = b.key where b.key = null`
* 右连接：`select * from table_a a right join table_b b on a.key = b.key where a.key = null`
* 全连接(MySQL不支持)：`select * from table_a a full outer join table_b b on a.key = b.key`
* 两种表单都没有数据的连接(MySQL不支持)：`select * from table_a a full outer join table_b b on a.key = b.key where a.key = null or b.key = null`

针对无法在MySQL使用的`full outer join`语句解决方法。

> 全连接，查询A、B的所有，使用左外连接拼上右外连接

```sql
SELECT
	*
FROM
	table_a a
LEFT JOIN table_b b ON a.key = b.key
UNION
	SELECT
		*
	FROM
		table_a a
	RIGHT JOIN table_b b ON a.key = b.key;
```

> 查询A、B的独有，左连接拼上右连接

```sql
SELECT
	*
FROM
	table_a a
LEFT JOIN table_b b ON a.key = b.key
WHERE
	b.key IS NULL
UNION
	SELECT
		*
	FROM
		table_a a
	RIGHT JOIN table_b b ON a.key = b.key
	WHERE
		a.key IS NULL;
```



### 3、索引是什么

* 索引是帮助Mysql高效获取数据的**数据结构**。

* 如果没有索引，查询某个数据，需要从头开始一个一个的查询，直到找到目标数据，而索引的出现可以直接定位到需要查找的数据的位置，提高了查找数据的效率。**索引就是排好序的快速查找数据结构。**

* 除了数据本身之外，数据库还维护着一个满足特定查找算法的数据结构，这种数据结构以某种方式指向数据，这样就可以在这些数据结构基础上实现高级查找算法，这种数据结构就是索引。

* 索引本身占内存也很大，所以是以索引文件的形式存储在磁盘上。

* 平常所说的索引，一般都是B+树结构的索引，除了B树索引还有hash索引。

* 索引的优势：
  * 提高检索效率，减少数据库的IO成本。
  * 通过索引对数据排序，降低了排序成本，减少CPU的销毁。

* 索引的劣势：
  * 索引也是一张表，需要占用空间。
  * 虽然索引提高了查询速度，但是会降低增删改的速度，每次都数据的修改，数据库不仅保存数据，还要保存变化后的索引文件。

### 4、索引的分类和命令

分类

* 单值索引：即一个索引只包含单个例，一个表可以有多个单值索引。
* 唯一索引：索引列的值必须唯一，但允许有空值。
* 复合索引：即一个索引包含多个列

命令

* 创建：`CREATE [UNIQUE] INDEX indexName ON mytable(表中字段)`或`ALTER mytable ADD [UNIQUE] INDEX [indexName] ON(表中字段)`
* 删除：`DROP INDEX [indexName] ON mytable`
* 查看：`SHOW INDEX FROM table_name`



### 5、索引的使用规则

* 需要创建索引的情况
  * 主键自动建立唯一索引
  * 频繁作为查询条件的字段应该创建索引
  * 与其他表关联的字段，外键关系建立索引
  * 频繁更新的字段不创建索引
  * where条件里用不到的字段不创建索引
  * 使用查询中排序的字段做索引，提高排序速度
  * 查询中统计或分组的字段
* 不需要创建索引的情况
  * 表记录太少
  * 经常增删改的表
  * 数据重复且平均的表字段

### 6、性能分析

Explain（查询执行计划），使用 Explain 关键字可以模拟 MySQL 优化器执行 SQL 语句，从而知道MySQL如何处理你的SQL语句，分析查询语句或是表结构的性能瓶颈。

Explain的功能：

* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询

使用Explain + SQL语句会显示如下信息：

![image-20200920224353870](http://picture.youyouluming.cn/image-20200920224353870.png)

表头信息解释：

* id：select 查询的序列号，包含一组数字，表示查询执行 select 子句或操作表的属性
  * id相同，执行顺序由上到下
  * id不同，如果是子查询，id的序号会递增，id值大的优先级高，先执行。
  * id有相同的也有不同的，id相同的是一组
* select_type：查询的类别，用于区别普通查询、联合查询、子查询等的复杂查询。
  * SIMPLE：简单的 select 查询，不包含子查询和UNION
  * PRIMARY：查询中若包含任何复杂的子查询，最外层则被标记为 PRIMARY
  * SUBQUERY：在 select 或 where 语句中的包含的子查询。
  * DERIVED：在 from 列表中包含的子查询被标记为DERIVED（衍生），MYSLQ 会递归执行这些子查询，把结果放在临时表。
  * UNION：若第二个 select 出现在 UNION 之后，则被标记为UNION
  * UNION RESULT：从 UNION 表获取结果的 select
* table：显示这一行数据是关于哪些表的。
* type：访问类型排列。从好到坏 system ---> const ---> eq_ref ---> ref ---> range ---> index ---> All
  * system：表只有一行记录，等于系统表，平时不会出现。
  * const：表示通过索引一次就找到了，const 用于比较主键或是 unique 索引。如将主键置于 where 列表中，MySQL 就会把该查询转换为一个常量。
  * eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，通常是主键或唯一索引键扫描。
  * ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质也是一种索引访问，返回所有匹配某个单独值的行，也可能找到多个条件的行，属于查找和扫描的混合体。
  * range：只检索给定的范围，使用一个索引来选择行，常在 where 语句中的 between、<、>、in 中出现。不用扫描全部索引。
  * index：index与All的区别是，index只扫描索引树，索引文件比真实的数据小的多，虽然也是读取全表，但index是从索引中读取的，All是从硬盘中读。比如以主键 id 为检索对象。
  * all：遍历全表找到匹配的行。
* possible_key：显示可能应用到这张表中的索引，一个或多个。查询涉及字段上若存在索引，则被列出。但不一定实际查询使用。
* key：实际使用到的索引，如果为 null，则没有使用索引。查询中若使用了覆盖索引，则该索引只出现在 key 列表中。
* key_len：表示索引中使用的字节数，通过此列计算查询中使用的索引长度，在不损失精度下，越短越好。显示的长度并非实际长度，根据表的定义计算出的。
* ref：显示索引的哪一列被使用了。哪些列或常量被用于查找索引列上的值。
* rows：根据表表统计信息和索引选用情况，大致估算出找到所需记录要读取的行数。
* Extra：包含不适合在其他列显示，但十分重要的信息。
  * Using filesort （×）：表示 MySQL 对数据使用了一个外部的索引排序，而不是按照表内的顺序进行读取。
  * Using temporary（×）：使用了临时表保存中间结果。常见于排序 order by 和 分组查询 group by。
  * Using index（√）：表示相应的 select 操作中出现了覆盖索引，避免的访问了表的数据行。如果同时出现 Using where 表明索引被用来执行索引键值的查找。没有 Using where 表名索引用来读取数据，而非查找动作。
    * 覆盖索引：是 select 的数据列只有从索引中就能获得，不必读取数据行，也就是查询列被所建的索引覆盖。
  * Using where：表明使用了where过滤
  * Using join buffer：使用了连接缓存
  * impossible where：where 的值总是 false，不能获取用来获取任何元组。

### 7、索引单表优化




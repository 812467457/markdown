# MySQL高级

## 一、索引优化分析

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

### 7、索引优化

#### 7.1、单表

> SQL 

```sql
CREATE TABLE `article` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `author_id` int(10) unsigned NOT NULL,
  `category_id` int(10) unsigned NOT NULL,
  `views` int(10) unsigned NOT NULL,
  `comments` int(10) unsigned NOT NULL,
  `title` varbinary(255) NOT NULL,
  `content` text NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
```



> 问题：查询 category_id 为 1，且comments > 1 并且 view 最多的 article_id 

```sql
EXPLAIN SELECT
	a.id,
	a.author_id
FROM
	article a
WHERE
	comments > 1
AND category_id = 1
ORDER BY
	views DESC
LIMIT 1
```

> 结果：

![image-20200921203804497](http://picture.youyouluming.cn/image-20200921203804497.png)

> 分析：

type 是 All 即最坏的情况。Extra 是 Using filesort 也是最坏的情况。所以必须优化。

> 解决：

建立一个复合索引`CREATE INDEX idx_article_ccv ON article(category_id,comments,views)`

再次查询结果

![image-20200921215646816](http://picture.youyouluming.cn/image-20200921215646816.png)

可以发现 type 从 ALL 变为 range 了，也使用上了自己创建的索引，但是 Using filesort 还未解决，因为该查询存在一个范围查找，即range类型查询字段后面的索引无效。

改进后：

创建索引时避免把范围查找的字段加入索引，删除之前的索引`DROP INDEX idx_article_ccv ON article`，再重写创建索引`CREATE INDEX idx_article_cv ON article(category_id,views)`

执行结果：

![image-20200921221944913](http://picture.youyouluming.cn/image-20200921221944913.png)

这时再执行查询语句，就可以同时使用到检索和排序的索引了。



#### 7.2、双表

> SQL 

```sql
CREATE TABLE IF NOT EXISTS `class`(
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
`card` INT (10) UNSIGNED NOT NULL
);
CREATE TABLE IF NOT EXISTS `book`(
`bookid` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
`card` INT (10) UNSIGNED NOT NULL
);
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
......
 
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
......
```



> 问题：有一个左连接查询`EXPLAIN SELECT * FROM book b LEFT JOIN class c ON b.card = c.card;`

没有创建任何索引的情况

> 创建右表索引 `CREATE INDEX idx_class_card ON class(card)`

结果

![image-20200921231215582](http://picture.youyouluming.cn/image-20200921231215582.png)



> 删除右表索引并 创建左表索引 	`DROP INDEX idx_class_card ON class; CREATE INDEX idx_book_card ON book(card);`

结果

![image-20200921231433967](http://picture.youyouluming.cn/image-20200921231433967.png)



> 左右表不同索引的分析

可以看出在左连接时，在右表创建索引的优势比在左表好。因为在左连接时，右表是关键的地方，左表无论如何都会全部查询，而条件是针对右表的，所以把索引建立在右表比较好。

左连接加右表索引。

#### 7.3、三表

>  SQL，在两表基础上添加一个表

```sql
CREATE TABLE IF NOT EXISTS `phone`(
`phoneid` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
`card` INT (10) UNSIGNED NOT NULL
)ENGINE = INNODB;

INSERT INTO phone(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card)VALUES(FLOOR(1+(RAND()*20)));
......
```



> 三表关联的查询

```sql
EXPLAIN SELECT
	*
FROM
	class c
LEFT JOIN book b ON c.card = b.card
LEFT JOIN phone p ON b.card = p.card;
```



> 没有索引的结果

![image-20200922084129695](http://picture.youyouluming.cn/image-20200922084129695.png)



> 为两个右表添加索引

```sql
CREATE INDEX idx_phone_card ON phone(card);
CREATE INDEX idx_book_card ON book(card);
```



> 结果

![image-20200922084443893](http://picture.youyouluming.cn/image-20200922093727180.png)



总结：

* 在使用 join 语句时，永远使用小结果驱动大的结果集。
* 优先优化嵌套循环的内层循环
* 保证 Join 语句中被驱动表上 Join 字段已被索引



#### 7.4、索引失效

* 全值匹配最好
* 最佳左前缀
* 不在索引列上做任何操作，否则会导致索引失效而导致全表扫描
* 存储引擎不能使用索引范围条件最右边的列
* 尽量使用覆盖索引，减少 select *
* MySQL 在使用不等于 (!= 或 <>) 的时候无法使用索引，会导致全表扫描
* is、null、is not null 也无法使用索引
* 使用 like 以通配符开头的索引也会失效
* 字符串不加单引号索引失效
* 尽量少用 or，or 连接也会导致索引失效



##### 情况一：最佳左前缀

> SQL ：建表并添加一个复合索引，索引有三个字段name、age、pos

```sql
CREATE TABLE staffs(
id INT PRIMARY KEY AUTO_INCREMENT,
`name` VARCHAR(24)NOT NULL DEFAULT'' COMMENT'姓名',
`age` INT NOT NULL DEFAULT 0 COMMENT'年龄',
`pos` VARCHAR(20) NOT NULL DEFAULT'' COMMENT'职位',
`add_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT'入职时间'
)CHARSET utf8 COMMENT'员工记录表';
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('z3',22,'manager',NOW());
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('July',23,'dev',NOW());
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('2000',23,'dev',NOW());

ALTER TABLE staffs ADD INDEX index_staffs_nameAgePos(`name`,`age`,`pos`)
```

> 使用索引中的所有字段进行查询

```sql
EXPLAIN SELECT
	*
FROM
	staffs
WHERE
	`name` = "July"
AND age = 23
AND pos = "dev"
```

结果：

![image-20200922093727180](http://picture.youyouluming.cn/image-20200922094209403.png)

这种情况看着一切正常，索引都使用上了。

但是如果查询条件换一下

```sql
EXPLAIN SELECT
	*
FROM
	staffs
WHERE
	age = 23
AND pos = "dev"
```

结果为：

![image-20200922094209403](http://picture.youyouluming.cn/image-20200922084443893.png)

可以发现实际建立的索引没有使用到，违背了最佳左前缀法则，也就是创建的索引必须从左到右的按顺序使用到，索引才会生效。



> 在使用到索引的第一个和第三个字段的情况

```sql
EXPLAIN SELECT
	*
FROM
	staffs
WHERE
	name = "July"
AND pos = "dev"
```

结果：

![image-20200922102409918](http://picture.youyouluming.cn/image-20200922102409918.png)



虽然使用到索引了，但是只用到了第一个索引，第三个明明使用了固定值也没有用到索引，还是因为违背了最佳左前缀的原则。



##### 情况二：索引列使用方法函数等操作

使用`EXPLAIN SELECT * FROM staffs WHERE `name` = "july";`查询结果为

![image-20200922103437558](http://picture.youyouluming.cn/image-20200922104125283.png)



使用`EXPLAIN SELECT * FROM staffs WHERE left(name,4) = "july";`查询结果为

![image-20200922104125283](http://picture.youyouluming.cn/image-20200922103437558.png)



在索引列上使用了函数就会导致索引的失效。





##### 情况三：存储引擎不能使用索引范围条件最右边的列

使用`EXPLAIN SELECT * FROM staffs WHERE `name` = "july" AND age > 20 AND pos = "manager";`查询结果为：
![image-20200922111640249](http://picture.youyouluming.cn/image-20200922111640249.png)

该查询从age开始到后面的索引都没有用上。索引中范围条件右边的会失效。



##### 情况四：尽量覆盖索引

使用`EXPLAIN SELECT name,age,pos FROM staffs WHERE `name` = "july" AND age > 20 AND pos = "manager";`查询结果为

![image-20200922142634501](http://picture.youyouluming.cn/image-20200922142634501.png)





##### 情况五：like 以通配符开头的索引会失效

两边都有 '%' 的情况 `EXPLAIN SELECT * FROM staffs WHERE name LIKE "%july%"` 的结果

![image-20200922144950621](http://picture.youyouluming.cn/image-20200922144950621.png)



只有左边有 '%' 的情况 `EXPLAIN SELECT * FROM staffs WHERE name LIKE "%july"` 结果

![image-20200922144927036](http://picture.youyouluming.cn/image-20200922144927036.png)



只有右边有 '%' 的情况 `EXPLAIN SELECT * FROM staffs WHERE name LIKE "july%"` 结果

![image-20200922144852878](http://picture.youyouluming.cn/image-20200922151239375.png)





只有把 '%' 加在右边索引才没有失效，但是某些场景下必须使用 '%' 。

解决方法：

> 建表和索引

```sql
CREATE TABLE `tbl_user`(
`id` INT(11) NOT NULL AUTO_INCREMENT,
`name` VARCHAR(20) DEFAULT NULL,
`age`INT(11) DEFAULT NULL,
`email` VARCHAR(20) DEFAULT NULL,
PRIMARY KEY(`id`)
)ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('1aa1',21,'a@163.com');
INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('2bb2',23,'b@163.com');
INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('3cc3',24,'c@163.com');
INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('4dd4',26,'d@163.com');

CREATE INDEX idx_user_nameAge ON tbl_user(`name`,age);
```

使用覆盖上索引避免全表扫描 `EXPLAIN SELECT name ,age FROM tbl_user WHERE `name` LIKE "%aa%"`  结果

![image-20200922151239375](http://picture.youyouluming.cn/image-20200922144852878.png)





## 二、查询截取分析

### 1、查询优化

#### 1.1、order by 关键字优化

> 建表和索引

```sql
create table tblA(
#id int primary key not null auto_increment,
age int,
birth timestamp not null
);

insert into tblA(age, birth) values(22, now());
insert into tblA(age, birth) values(23, now());
insert into tblA(age, birth) values(24, now());

create index idx_A_ageBirth on tblA(age, birth);
```



> 问题引出，观察以下 SQL 执行结果

![image-20200922223025192](http://picture.youyouluming.cn/image-20200922223025192.png)

![image-20200922223127491](http://picture.youyouluming.cn/image-20200922223127491.png)

![image-20200922223205875](http://picture.youyouluming.cn/image-20200922223205875.png)

![image-20200922223441704](http://picture.youyouluming.cn/image-20200922223441704.png)

> 解析

MySQL 支持两种方式排序，Using fIlesort 和 Using index，Index 的效率更高。

Order By 满足两个情况会使用 index 排序

1. Order by 语句使用索引最左前列
2. 使用 where 子句与 Order By 子句条件列组合满足索引最左前列

尽可能在索引列上完成排序操作，遵守最佳左前缀。无法使用索引时，增大 sort_buffer_size 参数的设置或增大 max_length_for_data 参数设置。



#### 1.2、Group by 关键字优化

和 order by 类似：

* group by 实质是先排序后进行分组，遵守索引最佳左前缀。

* 无法使用索引时，增大 sort_buffer_size 参数的设置或增大 max_length_for_data 参数设置。

* where 高于 having，能写 where 就不写 having。



### 2、慢查询日志

慢查询日志是 MySQL 提供的一种日志记录，用来记录在MySQL中相应时间超过阈值的语句，具体指运行时间超过 long_query_time 值的SQL，会被记录到慢查询日志中。long_query_time 默认值是10秒，可以手动设置。



> 查看、开启与设置

```sql
#查看
SHOW VARIABLES LIKE "%slow_query_log%";
#开启
SET GLOBAL slow_query_log = 1;
#查看慢查询的阈值
SHOW VARIABLES LIKE "%long_query_time%";
#设置阈值 3 秒，设置后再次查看没有变化，需要新的命令窗口查看才有变化
SET GLOBAL long_query_time = 3;
#查看当前系统有多少慢查询记录
SHOW GLOBAL STATUS LIKE "%slow_queries%";
```



> 测试与使用

使用 `select * sleep(4)` 制造一个慢查询，执行后来到记录日志的地方查看。

![image-20200923090626174](http://picture.youyouluming.cn/image-20200923090626174.png)



> 日志分析工具mysqldumpslow

使用 `mysqldumpslow --help;` 命令查看 mysqldumpslow，需要开启慢日志查询。

![image-20200923093156666](http://picture.youyouluming.cn/image-20200923093156666.png)



参数解释：

* s：表示按照何种方式排序
* c：访问次数
* i：锁定时间
* r：返回记录
* t：查询时间
* al：平均锁时间
* ar：平均返回记录数
* at：平均查询时间
* t：返回前面多少条数据
* g：后面搭配正则匹配模式

例如：

![image-20200923094055521](http://picture.youyouluming.cn/image-20200923155116127.png)





### 3、批量插入数据脚本

建表

```sql
CREATE TABLE `dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `deptno` mediumint(8) unsigned NOT NULL DEFAULT '0',
  `dname` varchar(40) NOT NULL DEFAULT '',
  `loc` varchar(40) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

CREATE TABLE `emp` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `empno` mediumint(8) unsigned NOT NULL DEFAULT '0' COMMENT '编号',
  `enmae` varchar(20) NOT NULL DEFAULT '',
  `job` varchar(20) NOT NULL DEFAULT '' COMMENT '工作',
  `mar` mediumint(8) unsigned NOT NULL DEFAULT '0' COMMENT '领导编号',
  `hiredate` date NOT NULL COMMENT '入职时间',
  `sal` decimal(7,2) NOT NULL COMMENT '薪水',
  `comm` decimal(7,2) NOT NULL COMMENT '红利',
  `deptno` mediumint(8) unsigned NOT NULL DEFAULT '0' COMMENT '部门编号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



先开启 bin_log，重启MySQL后失效

```sql
# 查看bin_log
SHOW VARIABLES LIKE "%log_bin_trust_function_creators%"
# 开启bin_log
SET GLOBAL log_bin_trust_function_creators = 1;
```



创建函数，保证每条数据不同

随机产生字符串的函数

```sql
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
	DECLARE char_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	DECLARE return_str VARCHAR(255) DEFAULT '';
	DECLARE i INT DEFAULT 0;
	WHILE i < n DO
	SET return_str = CONCAT(return_str,SUBSTRING(char_str,FLOOR(1+RAND()*52),1));
	SET i = i+1;
	END WHILE;
	RETURN return_str;
END $$
```

随机产生部门编号

```sql
DELIMITER $$
CREATE FUNCTION rand_num() RETURNS INT(5)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET i = FLOOR(100+RAND()*10);
	RETURN i;
END $$
```



创建存储过程

往emp表插入数据的存储过程

```sql
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;#把自动提交关闭
	REPEAT
	SET i = i + 1;
	INSERT INTO emp (empno,ename,job,mgr,hiredate,sal,comm,deptno) VALUES((START+i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
	UNTIL i = max_num
	END REPEAT;
	COMMIT;
END $$
```

往dept表插入数据的存储过程

```sql
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;
	REPEAT
	SET i = i + 1;
	INSERT INTO dept (deptno,dname,loc) VALUES((START+i),rand_string(10), rand_string(8));
	UNTIL i = max_num
	END REPEAT;
	COMMIT;
END $$
```



调用存储过程

```sql
# 向dept表插入10条数据
CALL insert_dept(100,10);
# 向emp表插入50w数据
CALL insert_emp(100001,500000);
```



### 4、使用 Show Profile 进行 SQL 分析

> profiling操作步骤

```sql
# 查看profiling状态
SHOW VARIABLES LIKE "%profiling%"

# 开启profiling
SET profiling = on;

# 查看执行SQL的记录
SHOW PROFILES;

# 诊断SQL，还有更多参数可以添加或删除，这里使用的是常用的参数。
SHOW PROFILE cpu,block io for query 上一步查询记录中的id
```

出现以下字段需要注意

![image-20200923155116127](http://picture.youyouluming.cn/image-20200923231858660.png)



## 三、MySQL锁机制

### 1、表锁（偏读）

偏向MyISAM存储引擎，开销小，加锁快，无死锁，锁定粒度大，发生锁冲突概率高，并发度最低。



> 建表

```sql
create table mylock(
id int not null primary key auto_increment,
name varchar(20)
)engine MyISAM;

insert into mylock(name) values('a');
insert into mylock(name) values('b');
insert into mylock(name) values('c');
insert into mylock(name) values('d');
insert into mylock(name) values('e');
```



> 锁的基本操作

```sql
# 查看哪些表有锁
SHOW OPEN TABLES;

# 为mylock表加读锁，为book表加写锁
LOCK TABLE mylock READ,book WRITE;

# 解锁
UNLOCK TABLES;
```



> 加读锁后的效果

```sql
# 给mylock加读锁
LOCK TABLE mylock READ;

SELECT * FROM mylock;

UPDATE mylock SET `name`="a2" WHERE id = 1; 

SELECT * FROM book;

UNLOCK TABLES;
```

结果：

查询当前加锁表可以。

更新当前加锁表失败。

查询其他没有加锁的表失败。

这时再启动一个终端对其加锁的表操作，可以查询，修改操作会被阻塞。

释放锁，另一个终端获得锁，执行成功。



> 加写锁后的效果

```sql
LOCK TABLE mylock WRITE;

SELECT * FROM mylock;

UPDATE mylock SET `name`="a2" WHERE id = 1; 

SELECT * FROM book;
```

结果：

可以查询当前表。

可以修改当前表。

无法查询其他没有加锁的表。

其他终端无法查询当前加锁的表，任何操作都会被阻塞。

释放锁，其他终端获得锁，执行成功。



> 总结：就是读锁会阻塞写操作，但是不会阻塞读操作，而写锁会阻塞读和写操作。



### 2、表锁分析

使用 `SHOW STATUS LIKE "table%";` 进行表锁分析

`SHOW STATUS LIKE "table%";` 的两个参数

* Table_locks_immediate：产生表级锁的次数，表示可以立即获取锁的查询次数，每次立即获取锁值加1。
* Table_locks_waited：出现表级锁定争用而发生等待的次数，不能立即获取锁的次数，每一次等待锁值加1。



MyISAM的读写锁调度是写优先，所以MyISAM引擎不适合做主表的引擎，因为写锁后，其他线程不能做任何操作，大量的查询很难得到锁，从而造成一直的堵塞。



### 3、行锁（偏写）

偏向InnoDB存储引擎，开销大，加锁慢，会出现死锁，锁定粒度最小，发生锁冲突的概率最低，并发度高。

InnoDB引擎和MyISAM引擎不同的是：InniDB支持事务，且是行级表。



> 建表

```sql
create table test_innodb_lock(a int(11),b varchar(16))engine=innodb;

insert into test_innodb_lock values(1,'b2');
insert into test_innodb_lock values(3,'3');
insert into test_innodb_lock values(4,'4000');
insert into test_innodb_lock values(5,'5000');
insert into test_innodb_lock values(6,'6000');
insert into test_innodb_lock values(7,'7000');
insert into test_innodb_lock values(8,'8000');
insert into test_innodb_lock values(9,'9000');
insert into test_innodb_lock values(1,'b1');

create index test_innodb_a_ind on test_innodb_lock(a);
create index test_innodb_lock_b_ind on test_innodb_lock(b);
```



> 行锁基本演示

终端A

```sql
# 关闭自动提交
SET autocommit = 0;

# 操作一
UPDATE test_innodb_lock SET b='4001' WHERE a = 4;

# 操作二
COMMIT;

# 操作三
UPDATE test_innodb_lock SET b='4002' WHERE a = 4;

# 操作四
COMMIT;

# 操作五
UPDATE test_innodb_lock SET b='4004' WHERE a = 4;

# 操作六
COMMIT;
```



终端B

```sql
# 关闭自动提交
SET autocommit = 0;

# 操作一、操作二
COMMIT;
SELECT * FROM test_innodb_lock;

# 操作三
UPDATE test_innodb_lock SET b = '4003' WHERE a = 4;
# 操作四
COMMIT;

# 操作五
UPDATE test_innodb_lock SET b='4005' WHERE a = 5;

#操作六
COMMIT;
```



结果：

终端A修改数据但未提交，终端B取查询数据会被阻塞。

终端A提交 数据，终端A查询成功。

终端A修改数据但未提交，终端B也去修改数据会被阻塞。

终端A提交，终端B执行成功并提交。

终端A修改a=4的数据，不提交。终端B 修改a=5的数据，没有被阻塞。



> 无索引行锁升级为表锁

终端A

```sql
# 这里的 b 是 varchar 类型的，此处故意不写引号，这样会自动类型转换，使索引失效
UPDATE test_innodb_lock SET a = 4 WHERE b = 4001;
```

终端B

```sql
UPDATE test_innodb_lock SET a = 9 WHERE b = 9000;
```



结果：

两个终端没有对同一行的数据进行修改，但是终端B执行后会进入阻塞状态，因为索引失效行锁自动转换为表锁了。



### 4、间隙锁的危害

什么是间隙锁：

当我们使用的是范围条件而不是相等条件检索数据，并请求共享数据排他锁时，InnoDB 会给符合条件的已有数据记录的索引项加锁，对于键值在条件范围内，但不存在记录的叫做间隙。

危害：

通过 Query 执行过程中通过范围查找了话，他会锁定整个范围内所有的索引键值，即使这个键值不存在。



问题描述：

一个线程对数据进行范围修改，然后索引不是连续的，另一个线程进行插入操作，插入的数据不在之前线程操作的范围内，最后插入的操作会进入阻塞。

![image-20200923231858660](http://picture.youyouluming.cn/image-20200923094055521.png)

端口A范围修改

```sql
UPDATE test_innodb_lock SET b = "1997" WHERE a > 1 and a < 6;
```

端口B新增数据

```sql
INSERT INTO test_innodb_lock VALUES (2,"2000");
```



解决方式：只锁一行

```sql
SELECT * FROM test_innodb_lock WHERE a = 8 FOR UPDATE
```

在SQL后面加上 FOR UPDATE 对当前操作的行上锁。其他操作会被阻塞，直到 COMMIT


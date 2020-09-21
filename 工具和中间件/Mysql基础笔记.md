# MySQL

## 一、SQL基本操作

### 1、基础查询

> 查询first_name和last_name，拼接成全名

```sql
SELECT
	CONCAT(first_name, last_name) "name"
FROM
	employees
```

CONCAT：SQL中字符串拼接的函数，**注意：其中有参数为null，拼接后全为null，使用函数IFNULL()解决。**

如：

```sql
SELECT
	CONCAT(
		first_name,
		last_name,
		IFNULL(commission_pct, "0") #该字段可能为空，使用ifnull(可能为null的字段,如果为null替换的数据)
	) "out_put"
FROM
	employees;
```



> 查询员工涉及到的部门编号，不要重复的部门编号

```sql
SELECT DISTINCT
	department_id
FROM
	employees
```

DISTINCT：去重查询



> 查询表结构

```sql
DESC employees;
# 或者
SHOW COLUMNS
FROM
	employees;
```



### 2、条件查询

> 查询工资大于15000的员工

```sql
SELECT
	first_name,
	salary
FROM
	employees
WHERE
	salary >= 15000;
```

where后面的条件可以使用关系运算符或逻辑表达式



> 查询部门编号不是100的员工信息

```sql
SELECT
	*
FROM
	employees
WHERE
	department_id <> 100
```



> 查询部门id不在50~100之间的员工信息

```sql
SELECT
	last_name,
	department_id,
	email
FROM
	employees
WHERE
	NOT (
		department_id >= 100
		AND department_id <= 50
	)
```



> 使用like关键词模糊查询，查询姓名包含字母a的员工信息

````sql
SELECT
	*
FROM
	employees
WHERE
	employees.last_name LIKE '%a%';
````



> in关键词，查询部门编号是30、50、90的员工信息

```sql
SELECT
	*
FROM
	employees
WHERE
	department_id IN (30, 50, 90);
```



> 使用between and，查询部门编号在30到90之间的员工信息

```sql
SELECT
	*
FROM
	employees
WHERE
	department_id BETWEEN 30 and 90
```



> 使用 is null 查询没有奖金的员工信息

```sql
SELECT
	*
FROM
	employees
WHERE
	commission_pct IS NULL
```



### 3、排序查询

> 将员工编号大于120的员工，按工资的升序排序

```sql
SELECT
	*
FROM
	employees
WHERE
	employee_id > 120
ORDER BY
	salary ASC
```

默认就是升序排序ASC，降序排序是DESC



> 使用别名，对有奖金的员工按年薪降序排序

```sql
SELECT
	*, salary * 12 * (1 + IFNULL(commission_pct, 0)) 年薪
FROM
	employees
WHERE
	commission_pct IS NOT NULL
ORDER BY
	年薪 DESC
```



> 使用函数的返回值进行排序，按照姓名的个数进行排序

```sql
SELECT
	LENGTH(last_name),
	last_name
FROM
	employees
ORDER BY
	LENGTH(last_name)
```



> 按多个字段进行排序，查询员工的信息，先按工资升序，再按部门编号降序。

```sql
SELECT
	*
FROM
	employees
ORDER BY
	salary ASC,
	department_id DESC;
```



## 二、SQL语法

### 1、内连接查询

> 查询部门编号大于100的部门名和所在城市名，SQL92语法

```sql
SELECT
	department_name,
	city
FROM
	departments d,
	locations l
WHERE
	d.location_id = l.location_id
AND d.department_id > 100
```



> 查询部门编号>100的部门名和所在城市名，SQL99语法

```sql
SELECT
	department_name,
	city
FROM
	departments d
JOIN locations l ON d.location_id = l.location_id
WHERE
	d.department_id > 100;
```



> 查询每个城市的部门个数

```sql
SELECT
	COUNT(*),
	city
FROM
	departments d
JOIN locations l ON d.location_id = l.location_id
GROUP BY
	l.city;
```



> 查询部门中员工个数>10的部门名，并按员工个数降序排名

```sql
SELECT
	COUNT(*) 员工人数,
	department_name
FROM
	departments d
JOIN employees e ON d.department_id = e.department_id
GROUP BY
	d.department_id
HAVING
	员工人数 > 10
ORDER BY
	员工人数 DESC;

```



> 查询员工名和对应的领导名

```sql
SELECT
	e.last_name,
	m.last_name
FROM
	employees e
JOIN employees m ON e.employee_id = m.manager_id;

```



### 2、外连接查询

> 查询哪个部门没有员工，并显示部门编号和部门名，左连接

```sql
SELECT
	d.department_id,
	d.department_name
FROM
	departments d
LEFT JOIN employees e ON e.department_id = d.department_id
WHERE
	e.department_id IS NULL;
```



### 3、子查询

>  查询工资比公司平均工资高到的员工信息，单行子查询

```sql
SELECT
	*
FROM
	employees
WHERE
	salary > (
		SELECT
			AVG(salary)
		FROM
			employees
	);
```



> 查询localtion_id是1400或1700的部门中的所有员工名字，多行查询

```sql
SELECT
	employees.last_name
FROM
	employees
WHERE
	department_id IN (
		SELECT DISTINCT
			department_id
		FROM
			departments
		WHERE
			location_id IN (1400, 1700)
	);
```






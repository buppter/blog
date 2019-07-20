---
title: MySQL 查询优化
date: 2019-07-20 16:46:36
categories: 数据库
tags: 
    - MySQL
    - 数据库
    - 数据库优化
---
上篇文章主要讲了对数据库索引的优化，除了对索引的优化外，查询优化也同样重要。这篇文章主要讲解对查询的优化。

<!--more-->

## 1. 永远小表驱动大表

类似嵌套循环
```python
for i in range(5):
    for j in range(1000):
        pass
        

for i in range(1000):
    for j in range(5):
        pass
        

# 第一种相当于小表驱动大表，我外层循环只需要执行五次
```


**即， 小的数据集驱动大的数据集**

```sql
# 实例SQL语句

SELECT * FROM A WHERE id IN (SELECT id FROM B)

# 等价于

for SELECT id FROM B
for SELECT * FROM A WHERE A.id = B.id
```

当B表的数据集小于A表的数据集是，用 `IN` 优于 `EXISTS`

---
```sql
SELECT * FROM A WHERE EXISTS (SELECT 1 FROM B WHERE B.id = A.id)

# 等价于

for SELECT * FROM A
for SELECT * FROM B WHERE B.id = A.id
```
当A表的数据集小于B表的数据集是，用 `EXISTS` 优于 `IN`

---

##### EXISTS
```sql
SELECT cloumn_list FROM table WHERE EXISTS (subquery)
```

该语法可以理解为：  
**将主查询的数据，放到子查询中做条件验证，根据验证结果( `TURE` 或 `FALSE` )来决定主查询的数据结果是否得以保留**

##### 提示

1. `EXISTS(subquery)` 只返回 `TURE` 或者 `FALSE` ，因此子查询的 `SELECT * ` 也可以是 `SELECT 1` 或 `SELECT 'X'` ,官方的说法是实际执行时会忽略 SELECT清单，因此没有区别
2. `EXISTS` 子查询的实际执行过程可能进过了优化而不是我们理解上的逐条对比，如果担心效率问题，可以进行实际检验确定是否存在效率问题。
3. `EXISTS` 子查询往往也可以用于条件表达式、其他子查询或 `JOIN` 来替代，何种最优需要具体问题具体分析。

## 2. IN 和 EXISTS 的具体实操

先看两张表的数据
```sql
mysql> select * from t_emp;
+----+------+--------+
| id | name | deptId |
+----+------+--------+
|  1 | n1   |      1 |
|  2 | n2   |      2 |
|  3 | n3   |      3 |
|  4 | n4   |      4 |
|  5 | n5   |      5 |
|  6 | n6   |     10 |
+----+------+--------+

mysql> select * from t_dept;
+----+----------+--------+
| id | deptName | locAdd |
+----+----------+--------+
|  1 | d1       | 11     |
|  2 | d2       | 12     |
|  3 | d3       | 13     |
|  4 | d4       | 14     |
|  5 | d5       | 15     |
|  6 | d6       | 16     |
+----+----------+--------+
```

具体的查询操作
 
![ZvSl0P.png](https://s2.ax1x.com/2019/07/19/ZvSl0P.png)

## 3. ORDER BY 优化

为排序使用索引

#### 1. MySQL的两种排序方式：文件排序(filesort)，扫描有序索引排序
#### 2. MySQL能为排序与查询使用相同的索引  

INDEX_key a_b_c(a, b, c)
```sql
    - ORDER BY a
    - ORDER BY a,b
    - ORDER BY a,b,c
    - ORDER BY a DESC, b DESC, c DESC 
```

**如果 `WHERE` 使用索引的最左前缀定义为常量，则 `ORDER BY` 能使用索引**
```sql
    - WHERE a=const ORDER BY b,c
    - WHERE a=const AND b=const ORDER BY c
    - WHERE a=const AND b>const ORDER BY c
```
**不能使用索引进行排序**
```sql
    - ORDER BY a ASC, b DESC, c DESC    /* 排序不一致*/
    - WHERE g=const ORDER BY b, c       /*丢失a索引*/
    - WHERE a=const ORDER BY c          /*丢失b索引*/
    - WHERE a=const ORDER BY a, d       /*d不是索引*/
    - WHERE a IN (...) ORDER BY b,c     /*对排序来说，多个相等条件也是范围查询*/
```
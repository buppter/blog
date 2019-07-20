---
title: MySQL 批量数据脚本
date: 2019-07-20 16:50:12
categories: 数据库
tags: 
    - MySQL
    - 数据脚本
---
使用脚本进行大数据量的批量插入，对特定情况下测试数据集的建立非常有用。

<!--more-->

## 一、建表

```sql
CREATE TABLE dept(
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '部门ID',
    dname VARCHAR(32) NOT NULL DEFAULT "" COMMENT '部门名称',
    loc VARCHAR(32) NOT NULL DEFAULT "" COMMENT '部门位置'
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE emp(
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '员工ID',
    ename VARCHAR(32) NOT NULL DEFAULT "" COMMENT "员工姓名",
    job VARCHAR(16) NOT NULL DEFAULT "" COMMENT "员工职位",
    mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT "上级编号",
    hiredata DATE NOT NULL COMMENT "入职时间",
    sal DECIMAL(7,2) NOT NULL COMMENT "薪水",
    comm DECIMAL(7,2) NOT NULL COMMENT "红利",
    deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT "部门编号"
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 二、 设置参数

如果创建函数的时候，报错
```shell
This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in ...
```
这是因为我们开启了 `bin-log`，我们必须为我们的 `function` 指定一个参数

先来查看当前的 `log_bin_trust_function_creators`

```sql
SHOW VARIABLES LIKE 'log_bin_trust_function_creators';

# 返回信息
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin_trust_function_creators | OFF   |
+---------------------------------+-------+
```

当前为 `OFF` 状态

开启 `log_bin_trust_function_creators`

```sql
SET GLOBAL log_bin_trust_function_creators = 1;
```
这样设置参数后，MySQL重启后参数会消失

## 三、创建函数，保证每条数据都不同

#### 1. 随机产生字符串
```sql
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
DECLARE return_str VARCHAR(255) DEFAULT '';
DECLARE i int DEFAULT 0;
WHILE i < n DO
SET return_str = CONCAT(return_str, SUBSTRING(chars_str, FLOOR(1 + RAND() * 52), 1));
SET i = i + 1;
END WHILE;
RETURN return_str;
END $$
```

`DELIMITER $$` , `END $$` 表示现在已 `$$` 表示一行 SQL 语句的完成，原来是 `;` ，现在继续用 `;` 就会发现已经失效，需要用 `$$` 来表示 SQL 语句的完成。
#### 2. 随机产生部门编号
```sql
DELIMITER $$
CREATE FUNCTION rand_num() RETURNS INT(5)
BEGIN
DECLARE i INT DEFAULT 0;
SET i = FLOOR(100 + RAND() * 10);
RETURN i;
END $$
```

## 四、创建存储过程

#### 1. 创建往 emp 表中插入数据的存储过程
```sql
DELIMITER $$
CREATE PROCEDURE insert_emp (IN START INT(10), IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
# SET autocommit = 0 把 autocommit 设置成 0 
SET autocommit = 0;
REPEAT
SET i = i + 1;
INSERT INTO emp(empno, ename, job, mgr, hiredata, sal, comm, deptno) VALUES ((START + i), rand_string(6), 'SALESMAN', 0001, CURDATE(), 2000, 400, rand_num());
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$
```
#### 2. 创建往 dept 表中插入数据的存储过程
```sql
DELIMITER $$
CREATE PROCEDURE insert_dept (IN START INT(10), IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
# SET autocommit = 0 把 autocommit 设置成 0 
SET autocommit = 0;
REPEAT
SET i = i + 1;
INSERT INTO dept(deptno, dname, loc) VALUES ((START + i), rand_string(10), rand_string(8));
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$
```

## 五、 调用存储过程

#### 1. dept
```sql
DELIMITER ;  /*恢复以 ;  表示语句的完成*/
CALL insert_dept(100,10);
```
#### 2. emp
执行存储过程，往 `emp` 表里添加50万数据
```sql
DELIMITER ;  
CALL insert_emp(100001,500000);
```
---
title: MySQL 慢查询日志
date: 2019-07-20 16:50:31
categories: 数据库
tags: 
    - MySQL
    - 慢查询日志
---
MySQL 的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过 `long_query_time` 值的SQL，则会被记录到慢查询日志中。 `long_query_time` 的默认值为 10，意思是运行 10s 以上的语句。默认情况下，MySQL 数据库并不启动慢查询日志，需要我们手动来设置这个参数，当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件，也支持将日志记录写入数据库表。

<!--more-->

## 一、 何为慢查询日志
> MySQL 的慢查询日志是MySQL提供的一种日志记录，它用来记录在 MySQL 中响应时间超过阈值的语句，具体指运行时间超过 `long_query_time`值的SQL
> `long_query_time` 的默认值为10，意思是运行超过十秒以上的语句

默认情况下，MySQL数据库没有开启慢查询日志，需要我们手动设置这个参数。

## 二、 如何查看和开启

```sql
# 查看数据库是否开启慢查询日志
SHOW VARIABLES LIKE '%long_query_log%';

# 返回信息
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_query_log      | OFF                                  |
| slow_query_log_file | /var/lib/mysql/8d0ed307ca92-slow.log |
+---------------------+--------------------------------------+

# 可以看到默认为关闭

# 开启慢查询日志
SET GLOBAL slow_query_log = 1;

# 再次查看，可以看到已经开启了慢查询日志
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_query_log      | ON                                   |
| slow_query_log_file | /var/lib/mysql/8d0ed307ca92-slow.log |
+---------------------+--------------------------------------+
```

`SET GLOBAL slow_query_log = 1;` 只对当前数据库生效，如果MySQL重启后则会失效。  

## 三、 开启慢查询后，什么样的 SQL 会被记录

这个是有参数 `long_query_time` 控制，默认情况下 `long_query_time` 值为 10 秒

查看 `long_query_time` 的值
```sql
SHOW VARIABLES LIKE 'long_query_time%';
```
如果运行时间刚好等于 `long_query_time` 的情况，并不会被记录下来，也就是说：  
在MySQL是**判断大于 `long_query_time` ，而不是大于等于**

## 四、 Case

#### 1. 设置 `long_query_time` 的值

```sql
SET GLOBAL long_query_time = 3;
```

#### 2. 修改了 `long_query_time` 的默认值后，为何不生效

通过命令将  `long_query_time` 的值设置为3秒，通过命令 `SHOW VARIABLES LIKE 'long_query_time%';` 却发现并没有生效 

**解决办法：**

1. 需要重新连接或新开一个会话才能看到修改的值
2. 通过命令 `SHOW GLOBAL VARIABLES LIKE 'long_query_time%';` 查看

#### 3. 记录慢查询日志并后续分析

为了测试，我们执行一条命令
```sql
SELECT SLEEP(4)
```
之后查看 `slow_query_log_file` 

```
mysqld, Version: 5.7.26 (MySQL Community Server (GPL)). started with:
Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
Time                 Id Command    Argument
# Time: 2019-07-19T04:11:40.523111Z
# User@Host: root[root] @ localhost []  Id:    36
# Query_time: 4.056847  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1563509500;
select sleep(4);
```

可以看到这条 SQL 语句已经被慢查询日志记录了下来，同时还有时间戳和具体的 SQL 语句以及其 `Query_time`,  `Lock_time`, `Rows_sent`, `Rows_examined`

#### 4. 查看当前系统中有多少条慢查询记录

```sql
SHOW GLOBAL STATUS LIKE '%Slow_queries%';

# 返回信息实例
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 1     |
+---------------+-------+
```

## 五、 日志分析工具-mysqldumpslow
> 在生产环境中，如果要手动分析日志，查找、分析 SQL ，显然是个体力活，MySQL提供了日志分析工具 `mysqldumpslow`


#### 1. mysqldumpslow 的帮助信息
```shell
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time  
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
```

参数| 参数解释|参数| 参数解释 |参数| 参数解释 |参数| 参数解释 
---|---|---|---|---|---|---|---
s| 表示按照何种方式排序| c | 访问次数 | r |返回记录|t|查询时间
al |平均锁定时间|ar|平均返回记录数|at|平均查询时间 |l|锁定时间
t |返回前面多少天的数据|g|后边搭配一个正则匹配模式大小写不敏感 

#### 2. 常用参考

```shell
# 得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/xxxxx-slow.log

# 得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/xxxxx-slow.log

# 得到按时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/xxxxx-slow.log

# 另外建议在使用这些命令是结合 | 和 more 使用，否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/xxxxx-slow.log | more
```
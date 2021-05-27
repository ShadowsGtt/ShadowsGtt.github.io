---
title: "[MySQL] MySQL学习记录"
search: false
author: "Shadow"
---


# MySQL零碎知识点

### MySQL全表扫描offer,limit优化

```
SELECT id FROM [table] WHERE id >= (SELECT id FROM [table] ORDER BY id LIMIT [offset] , 1) LIMIT [limit];
解释：offset为偏移量 limit为一次捞取的数据条数
id为表的自增主键
```

原理：offset limit的筛选方式是将数据全部提取出来后再丢弃不需要的数据，例如：`select * from table limit 1000,200 ;  ` 会捞取0 - 1200行的所有数据，然后再丢弃前1000行，返回所需200行。所以当表数据非常多时，捞取表末尾的数据自然就会变得很慢。 优化方法是加入子查询，子查询先将原本要获取的offset,limit区域的首行id找到，利用where条件直接跳到要查询的区域开始，然后再进行limit偏移。

### MySQL多表连接

#### 内连接:join、inner join



#### 外连接:left join、left outer join、right join、right outer join、union

#### 交叉连接:cross join



###mysqldump数据导出 -- 指定条件

> 注意：mysqldump默认会锁表,需要加上 `--skip-opt` 指定不锁表导出 

```
mysqldump -h[IP] -P[PORT] -u[USER] -p'[PWD]' [DB] [TABLE]  --default-character-set=utf8  --where="[CONDITION]" --skip-opt > my_export_data.sql

例如：
mysqldump -h127.0.0.1 -P3306 -uroot -p'123456!' test_db test_table  --default-character-set=utf8 --skip-opt --where="name='xiaoming' limit 100;" > my_export_data.sql

如果要将数据导入到其他地方需要注意：导出来的数据以sql形式保存，该sql文件会先drop表，然后创建一个新的表，然后将数据全部insert进入。
```

### 查询json中的某个字段

```
SELECT REPLACE(json_extract(COLOUMN_NAME,'$.KEY1.KEY2.KEY3'),'"','') FROM DATA_BASE;
// REPLACE函数去除结果中的双引号
```

### 为json中的字段添加索引

```
// test_table表中的json列事例数据如下：
{
	"info": {
		"name": "jack",
		"age": 12
	}
}
```

```
// 给 info 中的 name 添加索引（创建虚拟列）
alter table test_table
    add column info_name varchar(20) GENERATED ALWAYS 
    AS (json_extract(json_coloumn, '$.info.name')) VIRTUAL;
```



### 查看每个IP的连接数

```
select substring_index(host,':',1) as ip , count(*) from information_schema.processlist group by ip;
```



### Kill耗时最大的sql

```
// 显示用户正在运行的线程
show processlist;

// 查看最耗时的线程 加上 limit 5 表示查看 top 5
select * from information_schema.processlist where Command != 'Sleep' order by Time desc limit 5;

// 杀死耗时线程
kill [线程ID]

```

### 表备份

- 一条语句备份表

```
create table test_table_backup as select * from test_table;
```

上面语句可能会执行失败

ERROR 1786 (HY000): Statement violates GTID consistency: CREATE TABLE ... SELECT.

原因是:MySQL  5.6+之后只要开启了`ENFORCE_GTID_CONSISTENCY=on; `时候，MySQL只允许能够保证事物安全和能够被日志记录的SQL语句执行。像`create table ··· select···`的SQL语句以及**同时更新事务表和非事务表的SQL语句或事务都不允许执行**

- 如果一条语句的表备份方式失败了 那就分解一下

  第一句：`create table test_table_backup like table_backup;`	

  第二句：`SELECT * INTO test_table_backup from test_table; `

  ​				或者 `insert into test_table_backup select * from test_table;`



### 查看DB占用空间

```
select 
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024/1024, 2) as '数据容量(GB)',
truncate(index_length/1024/1024/1024, 2) as '索引容量(GB)'
from information_schema.tables
where table_schema='cc_account_event'
order by data_length desc, index_length desc;
```



# 日志文件binlog 

## 常用操作

- 查看`binlog`配置信息：`show variables like '%log_bin%';`
- 查看`binlog`格式： `show global variables like "binlog_format";`
- 查看`binlog`行镜像：`show global variables like 'binlog_row_image';`
- 查看`binlog`文件列表：`show binary logs;`
- 查看`binlog`文件状态信息：`show master status;`
- 清空`binlog`日志文件：`reset master;`
- 解析查看`bingo`文件内容：`show binlog events in 'mysql-bin.xxx' from [位置] limit [行数]\G`
- 修改`binlog`格式：`set global binlog_format='ROW/STATEMENT/MIXED';` 加`global`表示修改全局变量
- 下载远程`binlog`文件到本地： `mysqlbinlog -u[$User] -p[$Password] -h[$Host] --read-from-remote-server mysql-bin.xxx > [$File_Name]`
- 在客户端执行如下命令，通过mysqlbinlog工具查看Binlog日志文件内容：`mysqlbinlog -vv --base64-output=decode-rows mysql-bin.xxx | more`
- 

## 

# 慢查询



- 查看慢查询是否开启

  `show variables like '%slow_query%';`

- 查看有多少条慢查询记录

  `show global status like '%Slow_queries%';`

- 





# 事务

### 事务的四大特性

> 原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability），人们习惯称之为 ACID 特性
>
> - **原子性** （Atomicity）
>
>   事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。
>
> - **一致性** （Consistency）
>
>   事务开始前和结束后，数据库的完整性约束没有被破坏。例如:A和B账户总金额为500元，A给B转账100元，事务开始前他们的总金额为500，事务结束后他们的总金额还是500元。
>
> - **隔离性** （Isolation）
>
>   每个读写事务的对象对其他事务的操作对象能互相分离，即该事务提交前对其他事务不可见。
>
>   注：MySQL 通过锁机制来保证事务的隔离性。
>
> - **持久性** （Durability）
>
>   事务一旦提交，则其结果就是永久性的。即使发生宕机的故障，数据库也能将数据恢复，也就是说事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。
>
>   注：MySQL 使用 `redo log` 来保证事务的持久性。

### 查看事务隔离级别

```
第一种
SELECT @@global.tx_isolation, @@tx_isolation;   // mac下不支持

第二种
SHOW VARIABLES LIKE 'tx_isolation';
SHOW GLOBAL VARIABLES LIKE 'tx_isolation';

```



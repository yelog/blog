---
title: PostgreSQL事务及隔离级别
enlink: PostgreSQL事物及隔离级别
date: 2017-11-09 00:07:33
categories: 数据库
tags:
- PostgreSQL
---
### 介绍
PostgreSQL中提供了多种数据完整性的保证机制。如：约束、触发器、事务和锁管理等。

事务主要是为了保证一组相关数据库的操作能全部执行成功，从而保证数据的完整性。锁机制主要是控制多个用户对同一数据进行操作，使用锁机制可以解决并发问题。

### 事务
事务是用户对一个数据库操作的一个序列，这些操作要么全做，要么全不做，是一个不可分割的单位。

事务管理的常用语句如下：
```sql
BEGIN;
SQL语句1;
SQL语句2;
...
COMMIT;
```
事务块是指包围在BEGIN和COMMIT之间的语句。在PostgreSQL9中，常用的事务块管理语句含义如下：
>**START TRANSACTION**：此命令表示开始一个新的事务块.
**BEGIN**：初始化一个事务块。在BEGIN命令后的语句都将在一个事务里面执行，知道遇见COMMIT或ROLLBACK。它和START TRANSACTION是一样的。
**COMMIT**：提交事务。
**ROLLBACK**：事务失败时执行回滚操作。
**SET TRANSACTION**：设置当前事务的特性。对后面的事务没有影响。

### 事务隔离及并发控制
PostgreSQL是一个支持多用户的数据库，当多个用户操作同一数据库时，并发控制要保证所有用户可以高效的访问的同时不破坏数据的完整性。

数据库中数据的并发操作经常发生，而对数据的并发操作会带来下面的一些问题：

1. 脏读
一个事务读取了另一个未提交事务写入的数据。
2. 不可重复读
一个事务重新读取前面读取过的数据，发现该数据已经被另一个已经提交的事务修改。
3. 幻读
一个事务重新执行一个查询，返回符合查询条件的行的集合，发现满足查询条件的行的集合因为其它最近提交的事务而发生了改变。

SQL标准定义了四个级别的事务隔离。

| 隔离级别 | 脏读 | 幻读 | 不可重复性读取 |
| :- | :- |
|读未提交	|可能	|可能	|可能|
|读已提交	|不可能|	可能	|可能|
|可重复读	|不可能	|可能	|不可能|
|可串行读	|不可能	|不可能	|不可能|

在PostgreSQL中，可以请求4种隔离级别中的任意一种。但是在内部，实际上只有两种独立的隔离级别，分别对应已提交和可串行化。如果选择了读未提交的级别，实际上使用的是读已提交，在选择可重复读级别的时候，实际上用的是可串行化，所以实际的隔离级别可能比选择的更严格。这是SQL标准允许的：4种隔离级别只定义了哪种现象不能发生，但是没有定义哪种现象一定发生。

PostgreSQL只提供两种隔离级别的原因是，这是把标准的隔离级别与多版本并发控制架构映射相关的唯一合理方法。

1. 读已提交
这是PostgreSQL中默认的隔离级别，当一个事务运行在这个隔离级别时，一个SELECT查询只能看到查询开始前已提交的数据，而无法看到未提交的数据或者在查询期间其他的事务已提交的数据。
2. 可串行化
可串行化提供最严格的事务隔离。这个级别模拟串行的事务执行，就好像事务是一个接着一个串行的执行。不过，这个级别的应用必须准备在串行化失败的时候重新启动事务。

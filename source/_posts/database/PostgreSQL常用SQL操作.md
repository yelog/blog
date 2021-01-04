---
title: PostgreSQL常用SQL操作
enlink: postgres-sql-use
date: 2017-04-14 16:37:24
categories:
- 数据库
tags:
- postgres
- sql
---
>说明：文章中实例均在 `PostgreSQL` 环境操作。

## DDL数据定义语言
### 数据库/角色/schema
```sql
-- 创建一个数据库用户
create role "sp-boss" createdb createrole login password 'sp-boss';
-- 使用上面角色登录 postgres 数据库
psql -U sp-boss -d postgres
-- 创建自己的数据库
create database "sp-boss"
-- 登录自己的数据库
psql -U sp-boss
-- 创建一个其他用户
create role "sp-manager" login password 'sp-manager';
-- 赋予 create 权限
grant create on database "sp-boss" to "sp-manager";
-- 使用 新用户 登录数据库
psql -U sp-manager -d sp-boss
-- 创建自己的 schema
create schema "sp-manager";
```
### 表
```sql
--创建表
create table user_info (
  id serial primary key,
  name varchar(20),
  age integer,
  create_time timestamp,
  type integer,
  display boolean default true,
  unique (name, type)
);
--删除表
drop table exists user_info;
--重命名表
alter table user_info rename to user_infos;
```
### 字段（列）
```sql
--添加一列
alter table user_info add [column] username varchar(50);
--删除一列
alter table user_info drop [column] username;
--重命名列
alter table user_info rename [column] username to name;
--修改结构
alter table user_info alter [column] username set not null;
--
```
### 唯一约束
```SQL
-- 添加名为 uk_name 的联合唯一约束，组合列为column1和column2
alter table sys_theme add constraint uk_name unique(column1,column2);

-- 删除名为 uk_name 的约束
alter table sys_theme drop constraint uk_name;
```

## DML数据库操作语言
### SELECT
#### 查询包含json格式的text类型的数据
```sql
postgres=# select * from person;
 id |  name  |                          other                           
----+--------+----------------------------------------------------------
  1 | faker  | {"gender":"male","address":"xiamen","college":"xmut"}
  2 | watson | {"gender":"male","address":"shenzhen","college":"szu"}
  3 | lance  | {"gender":"male","address":"shenzhen","college":"xmut"}
  4 | jine   | {"gender":"female","address":"xiamen","college":"xmut"}
  5 | jobs   | {"gender":"male","address":"beijing","college":"xmu"}
  6 | yak    | {"gender":"female","address":"xiamen","college":"xmut"}
  7 | alice  | {"gender":"female","address":"shanghai","college":"thu"}
  8 | anita  | {"gender":"female","address":"xiongan","college":"hku"}
(8 行记录)
```
```sql
-- 查询深圳学生的高校分部情况
select
  other::json->>'college' college,
  count(1)
from
  person
where
  other::json->>'address'='shenzhen'
group by
  other::json->>'college';
___________________________
  college | count
 ---------+-------
  szu     |     1
  xmut    |     1
 (1 行记录)
--- 结果可得深圳一共有两个学生，
--- 在深圳大学和厦门理工学院各一个。
```

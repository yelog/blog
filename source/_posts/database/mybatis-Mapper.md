---
title: Mybatis常用Mapper语句
enlink: mybatis-Mapper
date: 2017-08-04 15:54:47
categories:
- 数据库
tags:
- mybatis
---

### 插入
```sql
/* 简单插入 */
<insert id="insertOne" parameterType="Person">
    insert into person (id, name, age) VALUES(#{id}, #{name}, #{age});
</insert>

/* 插入并返回对象的主键（数据库序列） */
<insert id="insertOne" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    insert into person (name, age) VALUES(#{name}, #{age});
</insert>
```

### 更新
```sql
/* 简单更新 */
<update id="updateName">
     update person set name = #{name} where id = #{id};
</update>

/* 更新值并返回 */
<select id="updateAge" parameterType="Person">
    update person set age = age + #{age} where id = #{id} returning age;
</select>
```

### 插入或更新
记录玩家在某种类型游戏下的统计记录：
>如果没有记录，则从插入，count字段为1；
如果有记录，则更新count字段+1；

1. 方式一
```sql
<insert id="addCount" parameterType="CountRecord">
    /*如果有记录，则更新；无记录，则noting*/
    update
      count_record
    set
      "count" = "count"+1
    where
      type_id = #{typeId}
    and
      user_id = #{userId};
    /*如果有记录，则noting；无记录，则插入*/
    insert into
      count_record(type_id, user_id, "count")
      select
        #{typeId}, #{userId}, 1
      where not exists
          (select
              *
           from
              count
           where
              type_id = #{typeId}
           and
              user_id = #{userId});
</insert>
```

2. 方式二
```sql
/* 利用 PostgreSQL 的 conflic 特性 */
<insert id="addCount" parameterType="CountRecord">
    insert into
      count_record(type_id, user_id, "count")
    VALUES
      (#{typeId}, #{userId}, #{count})
    on
      conflict(type_id,user_id)
    do update set
      "count" = count_record."count" + 1
</insert>
```

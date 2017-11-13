---
title: mysql过程循环结果集
date: 2017-11-13 23:30:36
desc: mysql过程中的循环
tags: mysql, procedure, loop
---

有个场景, 2张数据表, 表1模版表, 表2为具体实现表, 表2中有个字段是模版表的冗余字段, 现要求最快的方式实现数据同步!

<!--more-->

表1结构

```sql
    create table template (
        id int unsigned auto_increment,
        price decimal(8,2) not null default '0.0',

        primary key (id)
    ) engine=innodb charset=utf8 comment '模版表';

```

表2结构

```sql
    create table gifts (
        id int unsigned auto_increment,
        tid int unsigned not null default '0',
        price decimal(8,2) not null default '0.0',
        name varchar(32) not null default "" comment '名称',

        primary key (id)
    ) engine=innodb charset=utf8 comment "具体礼物表";
```




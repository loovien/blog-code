---
title: mysql过程循环处理结果集
date: 2017-11-13 12:57:10
desc: mysql在过程中处理查询的结果集
tags:  mysql, procedure
---









```sql

drop procedure if exists update_gifts_price;
delimiter $$
create procedure update_gifts_price ()
begin
    declare gid int;
    declare ttid int;
    declare tprice int;
    declare done int default false;
    declare c cursor for select id, tid from gifts;
    declare continue handler for not found set done = true;

open c;
read_loop: LOOP
    fetch c into gid, ttid;
    if done then
        leave read_loop;
    end if;
    select price into  tprice from gifts_template where id = ttid;
    select tprice, gid, ttid;
    update gifts set price = tprice where id = gid;
end LOOP;
close c;
end;
$$

delimiter ;

```

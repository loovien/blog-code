---
title: redis数据迁移
tags: []
date: 2015-07-26 02:40:00
---

##### 将数据从本地db1迁移到db2中 ###

很遗憾,redis只有[MOVE](http://redis.io/commands/move)命令,模式: MOVE KEY(需要移动的key) DB(到目标库中),只能单个键值移动,无法批量移动, 实现需要shell脚本

   `redis-cli -n 1 keys '*' | xargs -I '{}' redis-cli -n 1 move '{}' 2`

命令解析
   * redis-cli -n 1 keys '\*' 表示获取db1中所有的keys
   * xargs -I '{}' 表示讲上部操作的结果作为参数,保存在'{}'中, 在执行移动命令: redis-cli -n 1 move '{}' 2 

##### 将数据从本地服务器迁移到另外一台服务器上 ###

很遗憾,也是没有批量处理,只有[MIGRATE](http://redis.io/commands/migrate)命令, MIGRATE HOST(目标主机) PORT(目标端口) KEY(需要移动的key) DISTINATION_DB(目标库中) [COPY(拷贝本地数据,不删除)| REPLACE(覆盖目的主机上的键值) | 不填就等于复制本地后在删除本地数据 ] 只能单个移动,实现

   `redis-cli -n 1 keys '*' | grep regex | xargs -I '{}' redis-cli -n 1 migrate 192.168.1.100 11211 '{}' 2 COPY`
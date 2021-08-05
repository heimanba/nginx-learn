## 前言
在业务要求比较高的数据库中，对于`delete`和`update`后面加上`limit 1`绝对是个好习惯，如果在删除总，第一天就是命中了删除行，假设SQL 中有`limit 1`;这时就直接`return`了，否则还会执行完全表才`return`
> 如果是清空数据表,建议直接使用truncate,效率上远高于delete,因为truncate并不走事务，不会锁表，更不会产生大量的日志写入日志文件；truncate table table_name 后立刻释放磁盘空间，并重置 auto_increment 的值。delete 删除不释放磁盘空间，但后续 insert 会覆盖在之前删除的数据上

## 优点
```
delete from table_name where sex = 1;
```
- 降低SQL写错代价，如果删错了，并不致命
- 避免长事务，delete执行时MYSQL会将所有涉及的`行锁`和`Gap`锁。所有的`DML`语句执行相关的会被锁住，如果删除量大，会直接影响业务无法使用。
- delete 数据量大时，不加 limit 容易把 cpu 打满，导致越删越慢

## delete limit使用
如果要删除前10000行数据，有三种方法可以做到
- 直接执行 delete from T limit 10000
- 在一个连接中循环执行20次，delete from T limit 500
- 在20个连接中同时执行 delete from T limit 500

#### 优缺点
- 第一种: 直接delete使得执行事务时间过长
- 第二种: 效率慢点，但是每次都是短事务，不会锁同一条记录
- 第三种: 效率虽高，但容易锁住同一条记录，发生死锁的可能性比较高

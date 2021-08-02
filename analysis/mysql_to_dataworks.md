## 背景
日常数据分析工作中，遇到`MySQL`全量同步到`ADB`情况，直接使用`Dataworks`的数据集成，会出现下面的问题
- `MySQL`原始表数据有删减，使用`replace into`模式，并没有机制处理这部分已经删除的记录
- 如果使用`preSql`功能，在同步开始前，执行一些SQL，会先把表trauncate清空，然后在全量同步，才能解决。但是在truncate表再同步的过程中，`Dataworks`表中数据是空的如果量级很大，业务会全部中断。

![images](https://img.alicdn.com/imgextra/i4/O1CN01h8DFwN1L8NMcONozh_!!6000000001254-2-tps-2078-652.png)


## 解决方案
使用两张表，原表数据先`truncate`同步到`t_xx_clean`表中，另外的数据通过`replace into`模式同步到`t_xx`表，然后`t_xx_clean`和`t_xx`两张表进行`diff`，删除多余数据

![flow](https://img.alicdn.com/imgextra/i4/O1CN01RKYncK1weYymr0hX3_!!6000000006333-2-tps-1084-402.png)

1. 配置一个`Dataworks`集成任务，将全量数据写到`t_xx_clean`表，使用`[preSql]`功能，写入前，执行`truncate t_xx_clean`先把`t_xx_clean`表清空，然后全量同步一遍
2. 再配置另一个集成任务，将`MYSQL`数据通过`replace into`模式同步到`ADB`线上服务表，同步后，实际`MYSQL`已经删除的数据还在，通过`[postSql]`允许在数据同步完成后再到`ADB`执行一些SQL
```
delete from t_xx where id not in (
    select id from t_xx_clean
)
```
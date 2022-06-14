---
title: Mysql自动化分区
author: AnNong
categories: MYSQL
top: true
img: /img/cover_3.jpg
summary: 支持自定义数据库、分区表名称、分区个数、按天、按月、按年的自定义分区脚本
tags:
  - SQL
  - MYSQL存储过程
  - MYSQL事件
abbrlink: 63f7645b
date: 2022-04-29 11:04:10
---
## 创建存储过程

```sql
#创建自定义存储过程
DELIMITER ||
-- 删除存储过程
drop procedure if exists auto_set_partitions ||
-- 必须保证相应数据库表中至少有一个手动分区
-- 参数databasename：创建分区的数据库
-- 参数tablename：创建分区的表的名称
-- 参数partition_number：一次创建多少个分区
-- 参数partitiontype：分区类型[0按天分区，1按月分区，2按年分区]
-- 参数gaps：分区间隔，如果分区类型为0则表示每个分区的间隔为 gaps天；
--       如果分区类型为1则表示每个分区的间隔为 gaps月
--            如果分区类型为2则表示每个分区的间隔为 gaps年
create procedure auto_set_partitions (in databasename varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,in tablename varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci, in partition_number int, in partitiontype int, in gaps int)
L_END:
begin     
    declare max_partition_description varchar(255) default '';
    declare p_name varchar(255) default 0;       
    declare p_description varchar(255) default 0;   
    declare isexist_partition varchar(255) default 0; 
 declare i int default 1;
  
 -- 查看对应数据库对应表是否已经有手动分区[自动分区前提是必须有手动分区]
    select partition_name into isexist_partition from information_schema.partitions where table_schema = databasename  and table_name = tablename limit 1;
    -- 如果不存在则打印错误并退出存储过程
    if isexist_partition <=> "" then
       select "partition table not is exist" as "ERROR";
       leave L_END;
    end if;
 
    -- 获取最大[降序获取]的分区描述[值]
    select partition_description into max_partition_description  from information_schema.partitions where table_schema = databasename  and table_name = tablename order by partition_description desc limit 1;
   
    -- 如果最大分区没有,说明没有手动分区,则无法创建自动分区
    if max_partition_description <=> "" then
       select "partition table is error" as "ERROR";
       leave L_END;
    end if;
 
    -- 替换前后的单引号[''两个引号表示一个单引号的转义]
    -- set max_partition_description = REPLACE(max_partition_description, '''', '');
     -- 或使用如下语句
     set max_partition_description = REPLACE(max_partition_description-1, '\'', '');
 
   -- 自动创建number个分区
    while (i <= partition_number) do
                 if (partitiontype = 0) then
                     -- 每个分区按天递增,递增gaps天
                     set p_description = DATE_ADD(FROM_DAYS(max_partition_description), interval i*gaps day); 
                 elseif (partitiontype = 1) then
                     -- 每个分区按月递增,递增gaps月
                     set p_description = DATE_ADD(FROM_DAYS(max_partition_description), interval i*gaps month); 
                 else 
                     -- 每个分区按年递增,递增gaps年
                     set p_description = DATE_ADD(FROM_DAYS(max_partition_description), interval i*gaps year);
                 end if;
                 -- 删除空格
                 set p_name = REPLACE(p_description, ' ', '');
                 set p_description = DATE_ADD(p_description, interval 1 day); 
                 -- 如果有横杆替换为空
          set p_name = REPLACE(p_name, '-', '');
                 -- 删除时间冒号
                 set p_name = REPLACE(p_name, ':', '');
                 -- alter table tablename add partition ( partition pname values less than ('2017-02-20 10:05:56') );
          set @sql=CONCAT('ALTER TABLE ', tablename ,' ADD PARTITION ( PARTITION p', p_name ,' VALUES LESS THAN (TO_DAYS(\'', p_description ,'\')))');
          -- select @sql;
          PREPARE stmt from @sql;
          EXECUTE stmt;
          DEALLOCATE PREPARE stmt;
                 -- 递增变量
          set i = (i + 1) ;
 
    end while;          
end ||
-- 恢复语句中断符
DELIMITER ;
```

## 创建事件
```sql
# 创建事件
DELIMITER ||
drop event if exists auto_set_partitions  ||
create event auto_set_partitions 
on schedule every 1 day
starts '2021-07-02 15:03:10'
do
BEGIN
    call auto_set_partitions('interface_library', 'epidemic_report_result', 1, 0, 1);
END ||
DELIMITER ;
```



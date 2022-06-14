---
title: Mysql 各类日期函数的使用
author: AnNong
categories: MYSQL
img: /img/cover_2.png
summary: Mysql 查询今天，昨天，本月，今年，本季度，上一年等的sql语句
tags:
  - SQL
  - MYSQL
abbrlink: dce3f04e
date: 2022-04-27 15:08:11
---

- 今天
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERETO_DAYS(【时间字段名】) =TO_DAYS(now());
```

- 昨天
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE TO_DAYS( NOW( ) ) - TO_DAYS(【时间字段名】) = 1;
```
- 近七天
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE DATE_SUB(CURDATE(), INTERVAL 7 DAY) <=DATE(【时间字段名】);
```

- 本周内
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE YEARWEEK(DATE_FORMAT(【时间字段名】,'%Y-%m-%d')) = YEARWEEK(now());
```
- 上一周
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE YEARWEEK(DATE_FORMAT(【时间字段名】,'%Y-%m-%d')) = YEARWEEK(now())-1;
```

- 30天内 注意这个不是本月是从当天起向前推30天
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE DATE_SUB(CURDATE(), INTERVAL 30 DAY) <=DATE(【时间字段名】);
```

- 本月
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE DATE_FORMAT( 【时间字段名】, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' );
```

- 上一个月
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE PERIOD_DIFF( DATE_FORMAT( now( ) , '%Y%m' ) , DATE_FORMAT( 【时间字段名】, '%Y%m' ) ) =1;
```

- 本季度
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE QUARTER(【时间字段名】)=QUARTER(now());
```

- 上一季度
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE QUARTER(【时间字段名】)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));
```

- 本年度
```sql
SELECT 【想要的字段】 FROM 【表名】 WHERE YEAR(【时间字段名】)=YEAR(NOW());
```
- 上一年度
```sql
SELECT 【想要的字段】 FROM 【表名】 WHEREYEAR(【时间字段名】)=YEAR(date_sub(now(),interval 1YEAR));
```

> 日期函数说明：<br>
TO_DAYS() 将日期参数返回转换为天，给定一个日期date，返回一个日期号码（自0年以来的天数）。
NOW() 函数返回当前的日期和时间。
CURDATE() 函数返回当前的日期，是日期不是时间
DATE_SUB(current,INTERVAL 【N】 DAY) 将current向前推 N天
DATE_ADD(current,INTERVAL 【N】 DAY) 将current向后推 N天
YEARWEEK 是获取年份和周数的一个函数，函数形式为 YEARWEEK(date[,mode])
DATE_FORMAT( article_last_update, '%Y%m' ) 按照格式 格式化时间字符串
QUARTER(date) 返回日期的一年中的季度，范围为1到4。
YEAR(date) 返回日期的年份，范围为1000到9999，或者对于“零”日期返回0。
MONTH(date) 返回日期的月份，1月至12月的范围为1至12，对于包含月份为零的日期（如“0000-00-00”或“2008-00-00”），返回0。
WEEK(date[,mode]) 此函数返回日期的周号。 WEEK()的双参数使您能够指定星期是从星期天还是星期一开始，以及返回值是在0到53还是从1到53的范围内。如果省略mode参数，则值 使用了default_week_format系统变量。

 


---
title: Mysql分组统计最近七天数据，无则补零
author: AnNong
categories: MYSQL
cover: true
img: /img/cover_1.png
summary: Mysql根据日期自动分组统计最近七天最新数据，无则补零
tags:
  - SQL
  - MYSQL
abbrlink: 92d5164f
date: 2022-04-27 09:29:33
---

```java
SELECT
  DATE_FORMAT(a.timeDay, '%m-%d') AS time,
  IFNULL(b.success, 0) AS success,
  IFNULL(b.fail, 0) AS fail
FROM
  (
    SELECT
      curdate() AS timeDay
    UNION ALL
    SELECT
      date_sub(curdate(), INTERVAL 1 DAY) AS timeDay
    UNION ALL
    SELECT
      date_sub(curdate(), INTERVAL 2 DAY) AS timeDay
    UNION ALL
    SELECT
      date_sub(curdate(), INTERVAL 3 DAY) AS timeDay
    UNION ALL
    SELECT
      date_sub(curdate(), INTERVAL 4 DAY) AS timeDay
    UNION ALL
    SELECT
      date_sub(curdate(), INTERVAL 5 DAY) AS timeDay
    UNION ALL
    SELECT
      date_sub(curdate(), INTERVAL 6 DAY) AS timeDay
  ) a
  LEFT JOIN (
    SELECT
      date(d.last_execute_date) AS time,
      sum(
        CASE
          WHEN d.execute_status = '3' THEN 1
          ELSE 0
        END
      ) AS success,
      sum(
        CASE
          WHEN d.execute_status = '4' THEN 1
          ELSE 0
        END
      ) AS fail
    FROM
      data_collect d
    GROUP BY
      date(d.last_execute_date)
  ) b ON a.timeDay = b.time
ORDER BY
  time
```



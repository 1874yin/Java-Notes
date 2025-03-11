## MySQL 查询语句的执行顺序

一个经典的 SELECT 语句结构

```sql
SELECT DISTINCT column1, column2
FROM table1
JOIN table2 ON table1.id = table2.id
WHERE condition1 AND condition2
GROUP BY column3
HAVING condition3
ORDER BY column4
LIMIT 10;
```

#### 执行顺序

##### FROM

首先确定查询的表（FROM 子句），包括表的连接操作（JOIN）。这是查询执行的第一步，MySQL 会根据 `FROM` 子句中的表和连接条件（ON）来确定要操作的数据集。

如果有多个表，根据连接条件将它们组合成一个虚拟的中间结果集。

##### WHERE

在 `FROM` 子句生成的中间结果集上应用 `WHERE` 子句的过滤条件。`WHERE` 子句的作用是对中间结果集进行筛选。

`WHERE` 子句的条件是否使用索引会直接影响查询性能。

##### GROUP BY

对 `WHERE` 子句筛选后的结果集进行分组操作。`GROUP BY` 子句会将结果集按照指定的字段分组，并未每个分组生成一个聚合结果。

如果有 `GROUP BY` 但是没有聚合函数（SUM、COUNT），MySQL 会返回每个分组的第一条记录。

##### HAVING

在分组后的结果集上应用 `HAVING` 子句的过滤条件。`HAVING` 子句的作用是对分组后的结果进行进一步筛选。

`HAVING` 和 `WHERE` 的区别在于：`WHERE` 作用于单条记录， 而 `HAVING` 作用于分组后的聚合结果。

##### SELECT

从中间结果集中选择需要的字段，确定最终返回的字段。

如果使用了 `DINSTINCT` 关键字，会在这一阶段去除重复的记录。

##### ORDER BY

对最终的结果集进行排序。

操作排序非常耗时，尽量使用索引字段进行排序。

##### LIMIT

限制返回的结果集大小。


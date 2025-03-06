## 解决 MySQL 慢查询问题

#### 开启慢查询日志

步骤：

- 检查慢查询日志是否开启：

  ```sql
  SHOW VARIABLES LIKE '%slow_query%';
  SHOW VARIABLES LIKE '%long_query_time%';
  ```

- 开启慢查询日志：

  ```sql
  SET GLOBAL slow_query_log = 1;
  SET GLOBAL long_query_time = 1;
  ```

- 设置日志文件位置：

  ```sql
  SET GLOBAL slow_query_log_file = '/var/lib/mysql/slow-query.log';
  ```

#### 分析慢查询日志

使用 `mysqldumpslow`，MySQL 自带的工具分析查询日志。

```bash
# 查看最慢的10条SQL
mysqldumpslow -s t -t 10 /var/lib/mysql/slow-query.log
# 查看出现次数最多的10条SQL语句
mysqldumpslow -s c -t 10 /var/lib/mysql/slow-query.log 
```

#### 优化索引

索引是提高查询性能的关键。不合理的索引可能导致全表扫描，从而降低查询速度。

- 添加索引：对于经常出现在 `WHERE` 子句中的字段，考虑添加索引。

  ```sql
  ALTER TABLE orders ADD INDEX idx_user_time(user_id, create_time);
  ```

- 索引选择性：选择区分度高的列创建索引，避免使用区分度低的索引。

- 索引覆盖：确保查询所需的所有列都在索引中，避免回表查询。

#### 查询优化

- 避免 SELECT *：仅选择需要的列，减少数据传输量

  ```sql
  SELECT id, name FROM users WHERE status = 1;
  ```

- 减少子查询：尽量使用 JOIN 代替复杂的子查询。

- 利用 EXPLAIN：分析查询计划，识别全表扫描、未使用索引等问题。

  ```sql
  EXPLAIN SELECT * FROM users WHERE username = 'test';
  ```

- 分页优化：使用 LIMIT + OFFSET 分页时，考虑使用“延迟关联”或“覆盖索引+子查询”的技巧。

#### 数据库结构优化

优化表结构，提高查询效率。

- 归一化与反归一化：适度平衡数据冗余与查询性能。
- 分区表：针对大数据表，可以使用分区技术，按时间、范围或列表分区。

#### 系统配置调优

调整 MySQL 的配置参数可以提高数据库性能。

- 调整 `innodb_buffer_pool_size`：根据可用内存，合理设置 InnoDB 缓冲池大小。
- 调整 `max_connections`：限制最大连接数，防止连接耗尽。
- 调整 `query_cache_size`：对于读多写少的应用，适当增大查询缓存大小。

#### SQL模式调整

- 使用参数化查询：避免SQL注入风险的同时，有助于查询缓存的重用。
- 定期分析与优化表：使用 ANALYZE TABLE 和 OPTIMIZE TABLE 命令，更新统计信息，整理碎片。

#### 监控与报警

实时监控，及时发现并响应性能瓶颈。


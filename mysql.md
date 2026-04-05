# Performance guidelines for MySQL

### MySQL configuration

**innodb_buffer_pool_size**
- Storage area for caching data and indexes
- Should be bigger than the database size
- Recommended: 50-70% of total RAM
- Get current value (in MB) : `mysql> SELECT @@innodb_buffer_pool_size/1024/1024;`
- Set value (in bytes) : `mysql> SET GLOBAL innodb_buffer_pool_size=1073741824;`
- Set value in `my.ini` : `innodb_buffer_pool_size=1GB`
See https://dev.mysql.com/doc/refman/8.4/en/innodb-buffer-pool-resize.html

**innodb_log_file_size**
- Size of log files. Big log files mean better performance, but longer recovery time.
- Set value in `my.ini` : `innodb_log_file_size=256M`

**innodb_flush_log_at_trx_commit**
- `innodb_flush_log_at_trx_commit=0` write to buffer and flush to disk every second (fastest)
- `innodb_flush_log_at_trx_commit=1` write to buffer and flush to disk after each transaction (safest)
- `innodb_flush_log_at_trx_commit=2` write to buffer after each transaction and flush to disk every second (balance)

### Slow queries monitoring
```
# my.ini
# file to store slow queries information
slow_query_log=/var/log/mysql/slow-queries.log
# will log queries longer than 1s
long_query_time=1
# will log queries that are expected to retrieve all rows
log_queries_not_using_indexes=1
```
See https://dev.mysql.com/doc/refman/8.4/en/slow-query-log.html

### Indexes
- Unique values: `CREATE UNIQUE INDEX sku_idx ON product (sku);`
- Regular index: `CREATE INDEX sku_idx ON product (sku);` (for search by SKU)
- Descending index: `CREATE INDEX sku_idx ON product (sku DESC);` (speed up search in descending order)

### Misc
- Log optimization
- Indexes
- Avoid SELECT *
- Avoid sub-queries (`SELECT id, (SELECT name FROM b WHERE b.id=a.id) AS FROM a`)
- Lazy connection
- Table defragmentation
- Avoid using wildcard (`%`) in `LIKE` clause on indexes
- Use parameterized queries
- Avoid using `DISTINCT` when not necessary
- Use `LIMIT` for large result sets
- Use `JOIN` to solve N+1 query issue

### Rsources
- https://releem.com/blog/mysql-performance-tuning
- https://releem.com/blog/innodb-performance-tuning

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

**max_connections**
- Number of concurrent connections allowed to the database (default: 151)

### Cache
```
# my.ini
# enable cache (or disable it in case of high write/read rate)
query_cache_type = 1
# size of the cache
query_cache_size = 32M
# max size for a single cached query
query_cache_limit = 2M
```

**Monitor cache hits**
- `SHOW STATUS LIKE 'Qcache%';`  
- Cache hit ratio = hits / (hits + inserts) x 100 (should be over 80%)
- Cache Memory Usage should be below 70% of limit

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
- Use indexes on columns used in WHERE, JOIN, and ORDER BY

### Optimize SQL queries
- Avoid SELECT *
- Use `LIMIT` for large result sets
- Use `JOIN` to solve N+1 query issue
- Use parameterized queries
- Avoid using wildcard (`%`) in `LIKE` clause on indexes
- Avoid using `DISTINCT` when not necessary
- Avoid sub-queries (`SELECT id, (SELECT name FROM b WHERE b.id=a.id) AS FROM a`)

### Misc
- Lazy connection
- Table defragmentation
- Reuse catabase connections

### Resources
- https://releem.com/blog/mysql-performance-tuning
- https://releem.com/blog/innodb-performance-tuning
- https://releem.com/docs/mysql-performance-parameters

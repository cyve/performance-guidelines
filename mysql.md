# Performance guidelines for MySQL

- Log optimization
- Indexes
- Avoid SELECT *
- Avoid sub-queries (`SELECT id, (SELECT name FROM b WHERE b.id=a.id) AS FROM a`)
- Lazy connection
- Table defragmentation
- Slow queries monitoring
- Avoid using wildcard (`%`) in `LIKE` clause on indexes
- Use parameterized queries
- Avoid using `DISTINCT` when not necessary
- Use `LIMIT` for large result sets
- Use `JOIN` to solve N+1 query issue
- Configure `innodb_buffer_pool_size` (it should be bigger than the database size)
  - Get current value (in MB) : `mysql> SELECT @@innodb_buffer_pool_size/1024/1024;`
  - Set value (in bytes) : `mysql> SET GLOBAL innodb_buffer_pool_size=1073741824;`
  - Set value in `my.ini` : `innodb_buffer_pool_size=1GB`

### Rsources
- https://releem.com/blog/mysql-performance-tuning

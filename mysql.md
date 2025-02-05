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

### Rsources
- https://releem.com/blog/mysql-performance-tuning

# API

- Use payload compression/minification
- Use filters and pagination
- Reduce response payload (selected fields, remove duplicates and empty values, use serialization groups)
- Use cache
- Optimize serialization
- Avoid N+1 queries (use "eager" or "lazy" loading)
- Use async logging
- Prehead connections (database connection pool, persistent connection)
- Reduce number of proxies
- Use JSON instead of XML if possible

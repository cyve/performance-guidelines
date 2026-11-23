# CI

### jobs
- Parallelize jobs

### Composer
- Cache `vendor` directory
- Cache Composer global cache directory (default: `~/.composer/cache`, use `composer config cache-dir` to find it)

### PHPStan
- Set `tmpDir` parameter, and cache it (https://phpstan.org/config-reference#caching)

### PHPUnit
- Use plaintext password hasher to speed up login and fixtures generation.
- Use in memory filesystem to speed up I/O.
- Use SQL transactions to rollback database to avoid recreate a new database for each test.
- Use Paratest to parallelize tests
- Disable code coverage if not necessary
- Use versioned SQL dump instead of fixtures
- Cache SQL dump or SQLite database
- Identify slow tests with `phpunit --log-otr /tmp/otr/run.xml`, then `otr-report slowest`

### Docker
- Use layers to build images

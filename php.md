# Performance guidelines for PHP

- [PHP-FPM](#php-fpm)
- [OPCache](#opcache)
- [JIT compilation](#jit)
- [Realpath cache](#realpath-cache)
- [Composer](#composer)
- [Logging](#logging)
- [Applicative cache](#applicative-cache)
- [Framework optimized for performance](#framework-optimized-for-performance)
- [Parallelisation/asynchronicity](#parallelisation-asynchronicity)
- [XDebug](#xdebug)
- [Generators](#generators)
- [Code](#code)

### PHP-FPM
Set PHP-FPM as handler for `.php` files in Apache configuration
```
# /etc/apache2/conf/httpd.conf
<IfModule proxy_fcgi_module>
    # Enable http authorization headers
    <IfModule setenvif_module>
        SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
    </IfModule>

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php8.0-fpm.sock|fcgi://localhost:9000"
    </FilesMatch>
</IfModule>
```
See https://httpd.apache.org/docs/2.4/en/mod/mod_proxy_fcgi.html

Configure PHP-FPM
```
# /etc/php/8.3/fpm/pool.d/www.conf

# The management mode. Set "static" for high traffic apps and "ondemand" for low traffic apps
pm=static

# The maximum number of child processes allowed to be spawned
pm.max_children=128

# The number of child processes to start when PHP-FPM starts
pm.start_servers=4

# The minimum number of idle child processes PHP-FPM will create
pm.min_spare_servers=2

# The maximum number of idle child processes PHP-FPM will allow
pm.max_spare_servers=4

#The idle time, in seconds, after which a child process will be killed
pm.process_idle_timeout=10s

#The number of requests each child process should execute before respawning
pm.max_requests=200
```
See https://www.php.net/manual/en/install.fpm.configuration.php#pm

### OPCache
```
; php.ini
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000

# prod only
opcache.preload=/path/to/project/config/preload.php
opcache.preload_user=www-data
opcache.validate_timestamps=0
```
See https://www.php.net/manual/fr/opcache.configuration.php

### JIT
```
; php.ini
opcache.enable=1
opcache.jit=tracing
opcache.jit_buffer_size=100M
opcache.jit=tracing
```
The Just-In-Time compilcation optimizes the CPU load and may have low impact on application with lot of i/o.
See https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.jit

### Realpath cache
```
; php.ini
realpath_cache_size=4096K
realpath_cache_ttl=600
```
See https://www.php.net/manual/fr/ini.core.php#ini.realpath-cache-size

### Composer
```
# composer.json
"config":{
    "optimize-autoloader": true,
    "classmap-authoritative": true,
    "apcu-autoloader": true
}
```
Use `composer install --no-dev --optimize-autoloader --classmap-authoritative --apcu-autoloader`  
Use `composer dump-autoload --no-dev --optimize --classmap-authoritative --apcu`  
See https://getcomposer.org/doc/articles/autoloader-optimization.md

### Logging
```
; php.ini
log_errors=1
error_log=syslog

# in prod only
display_errors=0
display_startup_errors=0
error_reporting=E_ALL & ~E_WARNING & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED
```
See https://www.php.net/manual/en/errorfunc.configuration.php#ini.log-errors

### Applicative cache
Use APC, Redis or Memcached to store CPU/memory intensive operations.  
See https://www.php.net/manual/en/book.apcu.php

### Framework optimized for performance
- https://www.slimframework.com
- https://phalcon.io
- https://frankenphp.dev

### Parallelisation/asynchronicity
- [Fibers](https://www.php.net/manual/en/language.fibers.php)
- [parallel](https://www.php.net/manual/en/book.parallel.php)
- [OpenSwoole](https://github.com/openswoole/swoole-src)

### Xdebug
Disable Xdebug in production (of course) but also in development environment or CI when you don't need it (ex: composer install, etc.) If you can't disable the PHP extension, you can run PHP without Xdebug by setting the Xdebug mode (`XDEBUG_MODE=off php app.php`, or `php -d xdebug.mode=off app.php`)

### Generators
Use a generator to read data from a file or a database reduces memory usage
```php
public function readCsvFile(string $filepath): iterable
{
    $file = new SplFileObject('data.csv');
    while ($file->valid()) {
        yield $file->fgetcsv();
    }
}

foreach (readCsvFile('data.csv') as $line) {
    // do stuff
}

public function readDatabase(PDO $pdo): iterable
{
    $statement = $pdo->prepare('SELECT id,username,roles FROM users');
    $statement->execute();
    while ($row = $statement->fetch(PDO::FETCH_OBJ)) {
        yield $row;
    }
}

foreach (readDatabase(new PDO('') as $row) {
    // do stuff
}
```

Use generators instead of arrays when it's possible
```php
function generateNumbers($limit): \Generator {
    for ($i = 1; $i <= $limit; $i++) {
        yield $i;
    }
}

foreach (generateNumbers(100) as $number) {
    echo $number."\n";
}
```
See https://www.php.net/manual/en/language.oop5.iterations.php

### Code
- Use native PHP functions when possible
- Use static methods when possible
- Use `===` instead of `==`
- Delegate work to the database (filtering, sorting, etc.)
- Avoid requests to external sources (database, filesystem, webservice) in `for` or `while` loops.
- Do not use `SELECT *`, avoid `JOIN`, and add `LIMIT` in SQL queries.
- Execute batch `INSERT` queries if possible.
- Establish database connection only if necessary
- Avoid string manipulations (search, concatenation, etc.)
- Use temporary files (with `tmpfile()`) to create large fils line by line
- Avoid cloning object and use references
- Stream HTTP response
- Use `SplFixedArray` instead of array
- Use `SplMinHeap` instead of `sort()`
- Avoid `file_get_contents()`, `file()` and any function reading a entire file in a variable
- Use [lazy objects](https://github.com/symfony/var-exporter#lazyghosttrait)
- Close or unset unused variables
- Optimize loop execution
- Use `foreach` instead of `for`
- Avoid using global variables
- Use database connection pooling to avoid excessive connections
- Use typed variales
- Use simple quotes instead of double
- Use enums instead of constants
- Use readonly properties to avoid useless multiple assignments
- Use `array_chunk()` to split large array into smaller ones
- Use `unset()` to remove unused variable
- Use `gc_collect_cycles()` to remove all unused variables
- Minimize varialbles serialization
- to continue...

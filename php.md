# Performance guidelines for PHP

- [PHP-FPM](#php-fpm)
- [OPCache](#opcache)
- [JIT compilation](#jit-compilation)
- [Realpath cache](#realpath-cache)
- [Sessions](#sessions)
- [Multiple pools](#multiple-pools)
- [Composer](#composer)
- [Logging](#logging)
- [Applicative cache](#applicative-cache)
- [Framework optimized for performance](#framework-optimized-for-performance)
- [Parallelisation/asynchronicity](#parallelisationasynchronicity)
- [XDebug](#xdebug)
- [Generators](#generators)
- [Database persistent connection](#database-persistent-connection)
- [Stateful PHP](#stateful-php)
- [Optimized functions](#optimized-functions)
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

# Mode "dynamic" (default).
# PHP-FPM will start `start_servers` processes at startup and will fork new processes on demand until `max_children` is reached.
# Recommended values:
# `start_servers: CPU cores x4
# `min_spare_servers`: CPU cores x2
# `max_spare_servers`: CPU cores x4
# `max_children`: (total memory - system used by the system) / memory used by a process
pm=dynamic
pm.start_servers=4
pm.min_spare_servers=2
pm.max_spare_servers=4
pm.max_children=8

# Mode "ondemand" (recommended for low traffic applications)
# PHP-FPM will start `start_servers` processes at startup and will fork new processes when requests are received until `max_children` is reached.
# Idle processes are killed after `process_idle_timeout` is reached.
# Recommended values:
# `start_servers`: CPU cores x4
# `max_children`: (total memory - system used by the system) / memory used by a process
pm=ondemand
pm.start_servers=4
pm.max_children=128
pm.process_idle_timeout=10s

# Mode "static" (recommended for high traffic applications)
# PHP-FPM will start `max_children` processes at startup.
# Recommended value for `max_children`: (total memory - system used by the system) / memory used by a process
pm=static
pm.max_children=128

# The number of requests each child process should execute before respawning to avoid memory leaks
pm.max_requests=200

# Log request that take more than 3s
slowlog = /var/log/php-fpm.slow.log
request_slowlog_timeout = 3s

# Restart PHP-FPM if more than 10 requests fail within 1 minute
emergency_restart_threshold = 10
emergency_restart_interval = 1m

# Wait 1s before actually killing a process after receiving a KILL signal
process_control_timeout = 1s
```
See https://www.php.net/manual/en/install.fpm.configuration.php

### OPCache
```
; php.ini
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000
opcache.interned_strings_buffer=32

# prod only
opcache.preload=/path/to/project/config/preload.php
opcache.preload_user=www-data
opcache.validate_timestamps=0
opcache.file_update_protection=0
opcache.save_comments=0 # /!\ will ignore annotations
```
**Optimising configuration**

Look at the "Zend OPcache" section in the "phpinfo" page :
- Increase `opcache.memory_consumption` if the "Free memory" metric is close to 0.
- Increase `opcache.max_accelerated_files` if the "Cached keys" metric is close to the "Max keys" metric.
- Increase `opcache.interned_strings_buffer` if the "Interned Strings Free memory" metric is close to 0.

See https://www.php.net/manual/en/opcache.configuration.php

### JIT compilation
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
See https://www.php.net/manual/en/ini.core.php#ini.realpath-cache-size

### Sessions
```
# php.ini
# Set the probability to trigger the garbage collector to 0 for all requests
# /!\ set a cronjob to run `session_gc()` periodically
session.gc_probability=0

session.save_handler=redis
session.save_path=unix:///var/run/redis.sock?persistent=1
# or
session.save_path=tcp://redis:6379?persistent=1
```
See 
- https://www.php.net/manual/en/session.configuration.php
- https://www.getpagespeed.com/server-setup/php/cleanup-php-sessions-like-a-pro#how-debian-did-it

### Multiple pools
```
# /etc/php/8.3/fpm/pool.d/www.conf
[www]
listen = 127.0.0.1:9000
pm=static
php_admin_value[memory_limit]=128M
```
```
# /etc/php/8.3/fpm/pool.d/admin.conf
[admin]
listen = 127.0.0.1:9001
pm=ondemand
php_admin_value[memory_limit]=512M
```
```
# /etc/apache2/conf/httpd.conf
<FilesMatch "\.php$">
    SetHandler "proxy:unix:/var/run/php/php8.3-fpm.sock|fcgi://localhost:9000"
</LocationMatch>
<LocationMatch "^/admin">
    SetHandler "proxy:unix:/var/run/php/php8.3-admin-fpm.sock|fcgi://localhost:9001"
</LocationMatch>

# or setup a load balancer between pools
<FilesMatch "\.php$">
    SetHandler "proxy:balancer://php-fpm-cluster/"
</LocationMatch>
<Proxy "balancer://php-fpm-cluster/">
    BalancerMember "unix:/var/run/php/php8.3-fpm.sock|fcgi://localhost:9000"
    BalancerMember "unix:/var/run/php/php8.3-www2-fpm.sock|fcgi://localhost:9001"
</Proxy>
```

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

### Parallelisation/asynchronicity
- [Fibers](https://www.php.net/manual/en/language.fibers.php)
- [parallel](https://www.php.net/manual/en/book.parallel.php)
- [PCNTL extension](https://www.php.net/manual/en/pcntl.example.php)
- [OpenSwoole](https://github.com/openswoole/swoole-src)
- [spatie/fork](https://github.com/spatie/fork)
- [ReactPHP](https://reactphp.org)
- [AMPHP](https://amphp.org)

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

### Database persistent connection
```php
$pdo = new PDO('mysql:host=localhost;dbname=app', $user, $pass, [PDO::ATTR_PERSISTENT => true]);
```
> Persistent connections are not closed at the end of the script, but are cached and re-used when another script requests a connection using the same credentials. The persistent connection cache allows you to avoid the overhead of establishing a new connection every time a script needs to talk to a database, resulting in a faster web application.

See https://www.php.net/manual/en/pdo.connections.php

### Stateful PHP
- Benefits: reuse state (ex: database connection), reduce bootstrap overhead
- Drawbacks: state management, horizontal scaling, cache

```php
$server = new \OpenSwoole\HTTP\Server('127.0.0.1', 9501);
$server->on('start', function (Server $server) {
    echo "Connection open: {$req->fd}\n";
});
$server->on('request', function (Request $request, Response $response) {
    $response->header('Content-Type', 'text/plain');
    $response->end('Hello World');
});
$server->start();
```

### Optimized functions
Use the full qualified name of the following functions (ex: `\count()`):
- `strlen()`
- `count()`
- `is_null()`, etc.
- `intaval()`, etc.
- `defined()`
- `call_user_func()`
- `in_array()`
- `array_key_exists()`
- `gettype()`
- `get_class()`
- `get_called_class()`
- `func_num_args()`
- `func_get_args()`

### Code
- Use native PHP functions when possible
- Use static methods when possible (`array_*` functions, SPL library)
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
- Use `SplFixedArray` or `array_fill()` when the size is known to reserve memory
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
- Use `array_chunk()` and `unset()` to split large array and space memory
- Use `unset()` to remove unused variable
- Use `gc_collect_cycles()` to remove all unused variables
- Minimize varialbles serialization
- Avoid external dependencies when possible
- use `implode()` instead of string concatenation
- use `strtr()` instead of `str_replace()` for multiple replacements
- use `str_contains()', str_starts_with()` and `str_ends_with()` instead of `preg_match()` if possible
- store regex patterns in variables to precompile them
- use `ctype_*()` instead of `preg_match()` if possible
- use `+` instead of `array_merge()`
- use `[]` instead of `array_push()` to add element to array
- use `array_multisort()` instead of `usort()` if possible
- use `strtotime()` instead of `DateTime::createFromFormat()` when parsing common date formats
- to continue...

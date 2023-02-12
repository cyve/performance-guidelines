# Performance guidelines for PHP

- [php-fpm](#php-fpm)
- [OPCache](#opcache)
- [Realpath cache](#realpath-cache)
- [Composer](#composer)
- [Logging](#logging)
- [Applicative cache](#applicative-cache)
- [Tools optimized for performance](#tools-optimized-for-performance)
- [XDebug](#xdebug)
- [Generators](#generators)
- [Code](#code)

### php-fpm
Set php-fpm as handler for `.php` files in Apache configuration
```
# /etc/apache2/conf/httpd.conf
<IfModule proxy_fcgi_module>
    # Enable http authorization headers
    <IfModule setenvif_module>
        SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
    </IfModule>

    <FilesMatch "\.(php|phar)$">
        SetHandler "proxy:unix:/run/php/php8.0-fpm.sock|fcgi://localhost"
    </FilesMatch>
    
    # Deny access to raw php sources by default
    <FilesMatch "\.(php|phar)$">
        Require all denied
    </FilesMatch>
</IfModule>
```
See https://httpd.apache.org/docs/2.4/en/mod/mod_proxy_fcgi.html

### OPCache
```
; php.ini
opcache.enable=1;
opcache.memory_consumption=256
opcache.max_accelerated_files=20000

# prod only
opcache.preload=/path/to/project/config/preload.php
opcache.preload_user=www-data
opcache.validate_timestamps=0
```
See https://www.php.net/manual/fr/opcache.configuration.php

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
Use APC, Redis or Memcache to store CPU/memory intensive operations.  
See https://www.php.net/manual/en/book.apcu.php

### Tools optimized for performance
- https://www.slimframework.com
- https://phalcon.io
- https://github.com/openswoole/swoole-src
- https://frankenphp.dev

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
See https://www.php.net/manual/en/language.oop5.iterations.php

### Code
- Avoid requests to external sources (database, filesystem, webservice) in `for` or `while` loops.
- Do not use `SELECT *`, avoid `JOIN`, and add `LIMIT` in SQL queries.
- Execute batch `INSERT` queries if possible.
- Establish database connection only if necessary
- Avoid string manipulations (search, concatenation, etc.)
- to continue...

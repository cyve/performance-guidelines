# Performance guidelines for PHP

### Use php-fpm
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

### Optimize realpath cache
```
; php.ini
realpath_cache_size=4096K
realpath_cache_ttl=600
```
See https://www.php.net/manual/fr/ini.core.php#ini.realpath-cache-size

### Optimize composer autoload
Add `composer dump-autoload --no-dev --classmap-authoritative` to your deployment script to creates a class map and prevents Composer from scanning the file system.  
See https://getcomposer.org/doc/articles/autoloader-optimization.md

### Optimize logging
- (log errors in syslog)

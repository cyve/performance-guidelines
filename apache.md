# Performance guidelines for Apache

- [Enable HTTP2](#enable-http2)
- [Remove .htaccess](#remove-htaccess)
- [Use cache](#use-cache)
- [Enable compression](#enable-compression)
- [Disable unused modules](#disable-unused-modules)
- [Avoid using mod_rewrite when possible](#avoid-using-mod_rewrite-when-possible)
- [Configure MPM](#configure-mpm)
- [Optimize logs](#optimize-logs)
- [Disable DNS lookup](#disable-dns-lookup)
- [Use Unix sockets instead of TCP](#use-unix-sockets-instead-of-tcp)
- [Other links](#other-links)
- [Resources](#resources)

### Enable HTTP2
Enable module `mod_http2`. The module `mod_php` is not compatible with HTTP2, please use PHP-FPM.
```
# httpd.conf
LoadModule http2_module modules/mod_http2.so
Protocols h2 http/1.1
# Protocols h2c http/1.1 en dev pour désactiver le chiffrement TLS
```
See https://httpd.apache.org/docs/2.4/en/howto/http2.html

### Remove .htaccess
- Use `<Directory>` section in the Apache configuration to set specific directives.
- Use `AllowOverride None` to disable configuration override in `.htaccess` files.
See https://httpd.apache.org/docs/2.4/howto/htaccess.html#when

### Use cache
Enable module `mod_headers`. Add `ETag` and `Cache-Control` headers on static responses.
```
<FilesMatch ".(html|xml|csv|css|js|ico|jpe?g|png|gif|svg|eot|ttf|otf|woff|woff2|pdf|txt)$">
  # generate etag from file size and last modification time
  FileETag MTime Size
  # one year timeout
  Header set Cache-Control "max-age=2592000, must-revalidate"
</FilesMatch>
```
See https://httpd.apache.org/docs/2.4/en/caching.html

Enable module `mod_expires`. Add `Expires` headers on static responses.
```
<FilesMatch ".(html|xml|csv|css|js|ico|jpe?g|png|gif|svg|eot|ttf|otf|woff|woff2|pdf|txt)$">
  ExpiresDefault "access plus 1 month"
  ExpiresByType text/css "access plus 1 year"
  ExpiresByType text/html "access plus 1 day"
</FilesMatch>
```
See https://httpd.apache.org/docs/2.4/en/mod/mod_expires.html

### Enable compression
Enable module `mod_deflate`.
```
<FilesMatch ".(html|xml|csv|css|js|ico|jpe?g|png|gif|svg|eot|ttf|otf|woff|woff2|pdf|txt)$">
  SetOutputFilter DEFLATE
</FilesMatch>
# or
AddOutputFilterByType DEFLATE text/plain text/html text/xml text/css text/javascript application/javascript application/json
# or
SetOutputFilter BROTLI_COMPRESS
DeflateCompressionLevel 5
AddOutputFilterByType BROTLI_COMPRESS text/plain text/html text/css text/javascript application/json
```
See https://httpd.apache.org/docs/current/en/mod/mod_deflate.html

### Disable unused modules
See https://httpd.apache.org/modules

### Avoid using mod_rewrite when possible
Use `Redirect` or `Alias` directives instead of `RewriteRule` when possible.  
See https://httpd.apache.org/docs/trunk/en/rewrite/avoid.html

### Configure MPM

#### mpm_prefork_module
- Apache pre-start processes until `MaxRequestWorkers` is reached.
- Each HTTP request is handled by a separate child process with a single thread.
- Pros: Each process is isolated, so a problem in one does not affect others.
- Cons: High memory usage. Not as scalable for high traffic.
- Use Case: stability and non-thread-safe applications.

```
# /etc/apache2/mods-enabled/mpm-worker.conf
<IfModule mpm_prefork_module>
  # Maximum number of processes (recommended: (Total RAM - Memory used by the system) / process size)
  ServerLimit 256

  # Maximum number of concurrent requests (recommended: (Total RAM - Memory used by the system) / process size)
  MaxRequestWorkers 256

  # Number of processes created at startup (recommended: 1 per CPU core)
  StartServers 5

  # Minimum number of idle processes
  MinSpareServers 5

  # Maximum number of idle processes
  MaxSpareServers 100
</IfModule>
```
See https://httpd.apache.org/docs/2.4/en/mod/prefork.html

#### mpm_worker_module
- The main process launches child processes until `ThreadsPerChild` is reached.
- Multiple child processes, each with multiple threads handling one HTTP request.
- Pros: More scalable than prefork, lower memory usage, better for handling many simultaneous connections.
- Cons: A crash in one thread can affect others in the same process. Requires all code/modules to be thread-safe.
- Use Case: high-traffic sites with thread-safe applications.

See https://httpd.apache.org/docs/2.4/en/mod/worker.html

#### mpm_event_module
- Similar to worker, but keep TCP conenctions alive

See https://httpd.apache.org/docs/2.4/en/mod/event.html

### Optimize logs
Use directives `LogLevel`, `LogFormat` and `CustomLog` to log only useful events and informations.
```
# do not log access to static files
SetEnvIf Request_URI "\.(html|xml|csv|css|js|ico|jpe?g|png|gif|svg|eot|ttf|otf|woff|woff2|pdf|txt)$" dontlog
CustomLog /var/log/apache/access.log combined env=!dontlog
```
See https://httpd.apache.org/docs/2.4/en/logs.html

### Disable DNS lookup
Add directive `HostnameLookups Off` to Apache configuration

### Use Unix sockets instead of TCP
```
# /etc/apache2/conf/httpd.conf
<IfModule proxy_fcgi_module>
    <FilesMatch \.php$>
        # SetHandler "proxy:fcgi://localhost:9000"
        SetHandler "proxy:unix:/run/php/php8.4-fpm.sock"
    </FilesMatch>
</IfModule>
```

### Other links
- https://httpd.apache.org/docs/2.4/en/misc/perf-tuning.html

### Resources
- https://medium.com/@sbuckpesch/apache2-and-php-fpm-performance-optimization-step-by-step-guide-1bfecf161534

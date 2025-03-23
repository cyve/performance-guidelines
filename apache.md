# Performance guidelines for Apache

- [Enable HTTP2](#enable-http2)
- [Remove .htaccess](#remove-htaccess)
- [Use cache](#use-cache)
- [Enable compression](#enable-compression)
- [Disable unused modules](#disable-unused-modules)
- [Avoid using mod_rewrite when possible](#avoid-using-mod_rewrite-when-possible)
- [Use MPM prefork](#use-mpm-prefork)
--[Optimize logs](#optimize-logs)
- [Disable DNS lookup](#disable-dns-lookup)
- [Other links](#other-links)
- [Resources](#resources)

### Enable HTTP2
Enable module `mod_http2`. The module `mod_php` is not compatible with HTTP2, please use PHP-FPM.  
See https://httpd.apache.org/docs/2.4/en/howto/http2.html

### Remove .htaccess
Use `<Directory>` section in the Apache configuration to set specific directives.
See https://httpd.apache.org/docs/2.4/howto/htaccess.html#when

### Use cache
Enalbe module `mod_headers`. Add `ETag` and `Cache-Control` headers on static responses.
```
<FilesMatch ".(html|xml|csv|css|js|ico|jpe?g|png|gif|svg|eot|ttf|otf|woff|woff2|pdf|txt)$">
  # generate etag from file size and last modification time
  FileETag MTime Size
  # one year timeout
  Header set Cache-Control "max-age=2592000, must-revalidate"
</FilesMatch>
```
See https://httpd.apache.org/docs/2.4/en/caching.html

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

### Use MPM prefork
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
See https://httpd.apache.org/docs/2.4/en/mod/mpm_common.html

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

### Other links
- https://httpd.apache.org/docs/2.4/en/misc/perf-tuning.html

### Resources
- https://medium.com/@sbuckpesch/apache2-and-php-fpm-performance-optimization-step-by-step-guide-1bfecf161534

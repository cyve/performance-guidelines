# Performance guidelines for Apache

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
```
See https://httpd.apache.org/docs/current/en/mod/mod_deflate.html

### Disable unused modules
See https://httpd.apache.org/modules

### Avoid using mode_rewrite when possible
Use `Redirect` or `Alias` directives instead of `RewriteRule` when possible.  
See https://httpd.apache.org/docs/trunk/en/rewrite/avoid.html

### Use prefork
See https://httpd.apache.org/docs/2.4/en/mod/prefork.html

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

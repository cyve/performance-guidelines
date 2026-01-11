### Apache
```
# /etc/apache2/httpd.conf
LoadModule status_module modules/mod_status.so
<Location /server-info>
  SetHandler server-info
  Require local
  Require ip 172 # Allow Docker IPs
</Location>
<Location /server-status>
  SetHandler server-status
  Require local
  Require ip 172 # Allow Docker IPs
</Location>
```
- Get server information at http://localhost/server-info
- Get server status at http://localhost/server-status
- Get server status with machine-readable format at http://localhost/server-status?auto

### PHP-FPM
```
# /etc/php/8.3/fpm/pool.d/www.conf
pm.status_path = /fpm-status
```
```
# /etc/apache2/mods-available/status.conf
<Location /fpm-status>
  ProxyPass unix:/run/php/php8.3-fpm.sock|fcgi://localhost/fpm-status
  Require local
  Require ip 172 # Allow Docker IPs
</Location>
```
- Get PHP-FPM status at http://localhost/fpm-status
- Get PHP-FPM status with Prometheus format at http://localhost/fpm-status?openmetrics

```php
# fpm-status.php
<?php echo json_encode(fpm_get_status());
```

- https://www.php.net/manual/fr/fpm.status.php

### MySQL
(WIP)

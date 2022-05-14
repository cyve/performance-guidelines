# Performance guidelines for Apache

- Disable allowoverride => https://symfony.com/doc/current/setup/web_server_configuration.html#apache-with-mod-php-php-cgi
- No .htaccess
- PHP-FPM => https://symfony.com/doc/current/setup/web_server_configuration.html#apache-with-php-fpm
- HTTP/2 => https://www.linuxbabe.com/ubuntu/enable-http-2-apache-ubuntu-16-04-17-10
- Varnish => https://symfony.com/doc/4.4/http_cache/varnish.html
- Log optimization
- HTTP cache (etag, max-age, etc.)
- HTTP compression => https://www.whatsmyip.org/http-compression-test/

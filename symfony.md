# Performance guidelines for Symfony

- [Configuration](#configuration)
- [Service container](#service-container)
- [Routing](#routing)
- [Serialization](#serialization)
- [Security](#security)
- [Commands](#commands)
- [Doctrine](#doctrine)
- [Twig](#twig)
- [Kernel](#kernel)
- [Cache](#cache)
- [Misc](#misc)
- [In test environment](#in-test-environment)
- [Other links](#other-links)

### Configuration
- Restrict the number of locales enabled to only generate the translation files actually used.
```yaml
# config/packages/translation.yaml
framework:
    enabled_locales: ['en', 'fr']
```
- [Dump the service container](https://symfony.com/doc/5.4/components/dependency_injection/compilation.html#dumping-the-configuration-for-performance) in one file in production environment
```yaml
# config/services.yaml
parameters:
    container.dumper.inline_factories: true
```
- Reduce the automatic discovery of configuration files in `Kernel.php` (avoid `/**`) to speed up the kernel boot in development environment. Configure [different environments in a single file](https://symfony.com/doc/5.4/configuration.html#configuration-environments) instead.
```diff
- $container->import('../config/{packages}/*.{php,xml,yaml,yml}');
+ $container->import('../config/my_package/*.yaml');
- $container->import('../config/services/*.{php,xml,yaml,yml}');
- $container->import('../config/services_'.$this->environment.'.{php,xml,yaml,yml}');
+ $container->import('../config/services.yaml');
```

### Service container
- Use [lazy services](https://symfony.com/doc/5.4/service_container/lazy_services.html) or [service subscribers](https://symfony.com/doc/5.4/service_container/service_subscribers_locators.html) to avoid to instantiate unused services (specially in event listeners/subscribers, normalizers, voters or Twig extensions).
- Avoid to execute code in class constructor to speed up the kernel boot.
- Avoid to call database, filesystem or webservices in event listeners/subscribers.
- [Preload services](https://symfony.com/doc/5.4/reference/dic_tags.html#container-preload) to speed up the precompilation ([OPcache](https://www.php.net/manual/en/opcache.installation.php) needs to be enabled).
- Exclude non-service classes from the configuration to avoid the container to be rebuilt in development environment if these files change.
```yaml
# config/services.yaml
services:
    App\:
        resource: '../src/'
        exclude: '../src/{DependencyInjection,Entity,Event,Exception,Message,Kernel.php}'
```

### Routing
- Use [route groups](https://symfony.com/doc/5.4/routing.html#route-groups-and-prefixes) with a prefix instead of repeating the path on all routes to speed up the router in development environment.

### Serialization
- Make [custom normalizers cacheable](https://symfony.com/doc/5.4/serializer/custom_normalizer.html#performance).
- Reduce the serialization groups to the minimum.

### Security
- Make [custom voters cacheable](https://symfony.com/doc/5.4/security/voters.html#the-voter-interface).

### Commands
- Use [lazy commands](https://symfony.com/doc/5.4/console/commands_as_services.html#lazy-loading) to avoid the kernel to load all commands when you execute only one.

### Doctrine
See [Doctrine performance guidelines](doctrine.md)

### Twig
- Avoid to use logical instructions (`for`, `if`, `macro`, etc.), array filters (`map()`, `merge()`, etc.) or complex string manipulation (`json_encode()`, `replace()`, etc.) in templates if possible. Doing it in PHP is faster.
- Use [lazy-loaded extensions](https://symfony.com/doc/5.4/templating/twig_extension.html#creating-lazy-loaded-twig-extensions) to speed up the kernel boot.
- Do not use `render()`, use [Edge Side Includes (ESI)](https://symfony.com/doc/5.4/http_cache/esi.html) to cache frequently used parts of the page and/or [hinclude](https://symfony.com/doc/5.4/templating/hinclude.html) to load content asynchronously instead.
- Avoid to call database, filesystem or webservices in ESIs.
- [Preload assets](https://symfony.com/doc/5.4/web_link.html) using HTTP/2 and server push to speed up page loading.
- Use [Symfony UX](https://ux.symfony.com/) JavaScript components to speed up page loading.

### Kernel
- Avoid sub-requests if not necessary.
- Avoid to execute listeners/subscribers on sub-request events.
- Use `kernel.terminate` event to execute asynchronous work.

### Cache
- Use [applicative cache](https://symfony.com/doc/5.4/cache.html) to cache CPU/memory intensive operations.
- Make your [HTTP responses cacheable](https://symfony.com/doc/5.4/http_cache.html#making-your-responses-http-cacheable).
- Use [Varnish](https://symfony.com/doc/5.4/http_cache/varnish.html) or [Symfony's reverse proxy](https://symfony.com/doc/5.4/http_cache.html#symfony-reverse-proxy).
- Use [CachingHttpClient](https://symfony.com/doc/5.4/http_client.html#caching-requests-and-responses) to cache HTTP responses.

### Misc
- Optimize [logging](https://symfony.com/doc/5.4/logging.html).
- Use Symfony Messenger to [execute code](https://symfony.com/doc/5.4/messenger.html), [run processes](https://symfony.com/doc/5.4/components/process.html#running-processes-asynchronously) or [send emails](https://symfony.com/doc/5.4/mailer.html#sending-messages-async) asynchronously.

### In test environment
- Set environment variable `APP_CACHE_DIR=/dev/shm/symfony/cache` to store the cache in shared memory and speed up I/O.
- Set environment variables `APP_DEBUG=0` and `XDEBUG_MODE=off` to disable profiling during tests.
- Use plaintext password hasher to speed up login and fixtures generation.
- Use in memory filesystem to speed up I/O.
- Use SQL transactions to reset database to avoid recreate a new database for each test.
- Use `fingers_crossed` [Monolog handler](https://symfony.com/doc/5.4/logging.html#handlers-that-modify-log-entries) to reduce the amount of log.

### Other links
- https://symfony.com/doc/5.4/performance.html
- https://symfony.com/doc/5.4/the-fast-track/en/21-cache.html

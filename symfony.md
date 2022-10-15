# Performance guidelines for Symfony

### Configuration
- Apply Symfony's [performance checklist](https://symfony.com/doc/current/performance.html).
- Apply Symfony's [cache configuration](https://symfony.com/doc/current/the-fast-track/en/21-cache.html).
- Reduce the automatic discovery of configuration files in `Kernel.php` (avoid `/**`) to speed up the application in development environment. Configure [different environments in a single file](https://symfony.com/doc/5.4/configuration.html#configuration-environments) instead.
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
- Avoid to use database, filesystem or webservices in event listeners/subscribers.
- [Preload services](https://symfony.com/doc/5.4/reference/dic_tags.html#container-preload) in OPCache.
- Exclude non-service classes from the configuration to avoid the container to be rebuilt in development environment if theses files change.
```yaml
# config/services.yaml
services:
    App\:
        resource: '../src/'
        exclude: '../src/{DependencyInjection,Entity,Event,Exception,Message,Kernel.php}'
```

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
- Use [lazy-loaded extensions](https://symfony.com/doc/5.4/templating/twig_extension.html#creating-lazy-loaded-twig-extensions) to speed up the kernel boot.
- [Preload assets](https://symfony.com/doc/5.4/web_link.html) using HTTP/2 and server push to speed up page loading.
- Use [Edge Side Includes (ESI)](https://symfony.com/doc/5.4/http_cache/esi.html) to cache frequently used parts of the page.

### Kernel
- Avoid sub-requests if not necessary.
- Avoid to execute listeners/subscribers on sub-request events.
- Use `kernel.terminate` event to execute asynchronous work.

### Misc
- Use [applicative cache](https://symfony.com/doc/5.4/cache.html).
- Optimize [logging](https://symfony.com/doc/5.4/logging.html).
- Make your [HTTP responses cacheable](https://symfony.com/doc/5.4/http_cache.html#making-your-responses-http-cacheable).
- Use [Varnish](https://symfony.com/doc/5.4/http_cache/varnish.html) or [Symfony's reverse proxy](https://symfony.com/doc/5.4/http_cache.html#symfony-reverse-proxy).

### In test environment
- Set environment variable `APP_CACHE_DIR=/dev/shm/symfony/cache` to store the cache in shared memory and speed up I/O.
- Set environment variables `APP_DEBUG=0` and `XDEBUG_MODE=off` to disable profiling during tests.
- Use plaintext password hasher to speed up login and fixtures generation.
- Use in memory filesystem to speed up I/O.
- Use SQL transactions to reset database to avoid recreate a new database for each test.

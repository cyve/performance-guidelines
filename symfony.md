# Performance guidelines for Symfony

- Performance checklist => https://symfony.com/doc/current/performance.html
- Caching checklist => https://symfony.com/doc/current/the-fast-track/en/21-cache.html
- HTTP/2 preload / server push => https://symfony.com/doc/4.4/web_link.html
- Doctrine cache => https://medium.com/@renaudjacquot/retour-dexp%C3%A9rience-sur-l-optimisation-des-performances-sur-symfony-865ae5e30342#f18f
- Log optimization => https://symfony.com/doc/4.4/logging.html
- Do not perform code in services constructor
- Avoid sub-requests
- Run kernel.request subscribers on master requests only
- Use kernel.terminate to delay work to the background
- Use lazy services and commands
- Use Doctrine eager/lazy loading
- Use Doctrine hydratation modes when possible
- Use Doctrine partial object/partial reference when possible
- Use app cache => https://symfony.com/doc/current/cache.html
- Use ESI
- Avoid injecting services into Listeners/Subscribers. Inject full Container or lazy services instead
- Avoid calling database, filesystem or webservices in kernel listeners/subscribers
- Cache normalizers => https://symfony.com/doc/current/serializer/custom_normalizer.html#performance
- Clear config files loading in Kernel
- Use lazy Twig extensions => https://symfony.com/doc/current/templating/twig_extension.html
- Preload services => https://symfony.com/blog/new-in-symfony-5-1-configurable-php-preloading
- Use ServiceSubscriber => https://symfony.com/doc/current/service_container/service_subscribers_locators.html
- https://jolicode.com/blog/battle-log-a-deep-dive-in-symfony-stack-in-search-of-optimizations-1-n
- https://jolicode.com/blog/battle-log-a-deep-dive-in-symfony-stack-in-search-of-optimizations-2-n

In dev envirnonment :
- `doctrine.dbal.profiling_collect_schema_errors: false`

In test environment :
- Set cache directory to /dev/shm/symfony/cache
- Use plaintext password hasher
- `doctrine.dbal.logging: false`
- Set APP_DEBUG to false
- XDEBUG_MODE=off php bin/phpunit

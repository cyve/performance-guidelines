# Doctrine performance guidelines

**Use the right fetch mode**
```php
/* @ORM\OneToMany(targetEntity="Item", fetch="EAGER") */
private Collection $items;
```
`LAZY` : relation will be loaded as reference (default)  
`EAGER` : relation will be fully loaded  
`EXTRA_LAZY` : allows to perform operations on collections (ex: `$items->contains(...)`) without loading the full data

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/annotations-reference.html#onetomany
- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/tutorials/extra-lazy-associations.html

**Use the right hydratation mode**
```php
$query = $em->createQuery('SELECT u FROM App\Entity\User u');
$users = $query->getResult(Query::HYDRATE_ARRAY);
```
`HYDRATE_OBJECT` : return object hydrated through reflection (default)  
`HYDRATE_SIMPLEOBJECT` : same as above but without relations  
`HYDRATE_ARRAY` : return deduplicated associative array  
`HYDRATE_SCALAR` : return raw SQL data  
`HYDRATE_SINGLE_SCALAR` : return scalar if the query result is a single scalar (ex: `SELECT COUNT(*) FROM user`)  
`HYDRATE_SCALAR_COLUMN` : return array of scalars if the query result is a column (ex: `SELECT p.id FROM user`)

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/dql-doctrine-query-language.html#hydration-modes

**Use partial objects when necessary**
```php
// return object with only `id` and `name` properties hydrated. The other properties are null.
$query = $em->createQuery("SELECT PARTIAL u.{id,name} FROM App\Entity\User u");

// return object with only identifier hydrated. The other properties are null.
$query = $em->getPartialReference(User::class, $id);
```

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/partial-objects.html

**Use batch processing**
```php
$batchSize = 10;
for ($i = 1; $i <= 1000; ++$i) {
    $user = new User;
    $user->setUsername('user'.$i);
    $em->persist($user);
    if (($i % $batchSize) === 0) {
        $em->flush();
        $em->clear(); // detach flushed objects
    }
}
$em->flush(); // persist remaining objects
$em->clear();
```

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/batch-processing.html

**Iterate on large results**
```php
$query = $this->em->createQuery('SELECT u FROM App\Entity\User u');
foreach ($query->toIterable() as $row) {
    // do stuff
    $this->em->detach($row[0]); // detach entity
}
```

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/batch-processing.html#iterating-large-results-for-data-processing

**Disable logging and profiling**
```yaml
doctrine:
  dbal:
    logging: "%kernel.debug%" // logging will be disabled if you set `APP_DEBUG=false` in `dev` or `test` env.
    profiling: "%kernel.debug%"
```

**Configure Doctrine cache**
```yaml
when@prod:
    doctrine:
        orm:
            metadata_cache_driver: // will cache the entities metadata in the "system" cache
                type: pool
                pool: doctrine.system_cache_pool
            query_cache_driver: // will cache the computed DQL queries in the "system" cache
                type: pool
                pool: doctrine.system_cache_pool
            result_cache_driver: // will cache the queries results in the "app" cache
                type: pool
                pool: doctrine.result_cache_pool
    framework:
        cache:
            system: cache.adapter.system
            app: cache.adapter.apcu
            pools:
                doctrine.result_cache_pool:
                    adapter: cache.app
                doctrine.system_cache_pool:
                    adapter: cache.system
```

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/caching.html
- https://symfony.com/doc/current/reference/configuration/doctrine.html#caching-drivers
- https://symfony.com/doc/current/cache.html

**Use readonly entities**
```php
/* @Entity(readOnly=true) */
class User {}
```
The object can only be persisted, read, detached and removed. Updates will be skipped during flush.
```php
// explicitally mark object as read only
$user = $em->find(User::class, $id);
$em->getUnitOfWork()->markReadOnly($user);

$users = $em->createQuery('SELECT u FROM App\Entity\User u')
  ->setHint(Query::HINT_READ_ONLY, true)
  ->getResult();
```

- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/improving-performance.html#read-only-entities
- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/annotations-reference.html#entity

**Best practices for performance optimizations**
- Avoid bidirectional relations
- Avoid using lifecycle events
- Avoid using `cascade`
- Avoid using quotes or special characters in table/column names
- Avoid composite identifiers
- Do not map foreign keys to fields in an entity
- Define transactions explicitally
- Initialize collections in the constructor

**Misc**
- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/improving-performance.html
- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/second-level-cache.html
- https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/best-practices.html

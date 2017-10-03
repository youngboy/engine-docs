---
title: Whole Query Caching
order: 3
---

Engine allows you to cache whole GraphQL query responses. You can use Memcached as the cache store. You can set TTLs per query signature.

These are the the steps to configure whole query caching:

1. Configure cache stores
2. Configure caching policies for operation signatures
3. Add the caching configuration to the Engine config JSON

### Configure cache stores

Engine supports one or more cache stores. Each cache store can consist of one or more Memcached instances or in-process caches.

Here's an example configuration entry for cache stores:

```
"stores": [
    {
      "name": "standardCache",
      "timeout": "1s",
      "memcaches": [
        {
          "url": "localhost:11211"
        }
      ]
    }
  ],
```

where:

* name: user-given id for the cache store
* timeout: the timeout, with units, for the Proxy to connect to Memcached
* [optional] memcaches: list of Memcache instances to be used for this cache store. Each Memcached configuration contains a Memcached URL. If this is omitted, an in-memory cache is created.

### Set TTL for a operation signature

Here's an example configuration entry for configuring TTL (in seconds) and a cache store for each operation signature:

```
"operations": [
    {
      "signature": "{hero{name}}",
      "caches": [
        {
          "ttl": "600s",
          "store": "standardCache"
        }
      ]
    }
  ],
```

### Invalidate a cache store

To invalidate a cache store, change the key prefix on the cache store configuration.

```
"stores": [
    {
      "name": "standardCache",
***      "memcacheKeyPrefix": "1",***
      "timeout": "1s",
      "memcaches": [
        {
          "url": "localhost:11211"
        }
      ]
    }
  ],

```

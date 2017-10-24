---
title: Response Caching
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
      "memcache": {
        "urls": [
          "localhost:11211"
        ],
        "timeout": "1s",
      }
    }
  ],
```

where:

* name: user-given id for the cache store
* timeout: the timeout, with units, for the Proxy to connect to Memcached
* [optional] memcaches: list of Memcache instances to be used for this cache store. Each Memcached configuration contains a Memcached URL. If this is omitted, an in-memory cache is created.

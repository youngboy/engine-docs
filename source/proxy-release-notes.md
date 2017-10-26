---
title: Proxy release notes
order: 20
---

### 2017.10-425-gdd4873ae

* Removed empty values in the request to server: `operationName`, `extensions`.
* Improved error message when handling a request with GraphQL batching. Batching is still not supported at this time.


### 2017.10-408-g497e1410

* Removed limit on HTTP responses from origin server.
* Fixed issue where `apollo-engine-js` would fail to clean up sidecar processes.
* Switched query cache compression from LZ4 to Snappy.
* *BREAKING*: Renamed the `logcfg` configuration section to `logging`.
* *BREAKING*: Nested HTTP/Lambda origin configurations under child objects: `http` and `lambda`.
* Added HTTP request logging, and GraphQL query logging options.

These changes mean that a basic configuration like:

```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  },
  "origins": [
    {
      "url": "http://localhost:3000/graphql"
    }
  ],
  "frontends": [
    {
      "host": "127.0.0.1",
      "port": 3001,
      "endpoint": "/graphql"
    }
  ]
}
```

Is updated to:

```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logging": {
    "level": "INFO"
  },
  "origins": [
    {
      "http": {
        "url": "http://localhost:3000/graphql"
      }
    }
  ],
  "frontends": [
    {
      "host": "127.0.0.1",
      "port": 3001,
      "endpoint": "/graphql"
    }
  ]
}
```


### 2017.10-376-g0e29d5d5

* Added (debug) log message to indicate if a query's trace was selected for reporting.
* Fixed an issue where non-GraphQL errors (i.e. a `500` response with an HTML error page) would not be tracked as errors.

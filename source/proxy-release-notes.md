---
title: Proxy release notes
order: 20
---

### 2017.11-40-g9585bfc6

* Fixed a bug where query parameters would be dropped from requests forwarded to origins.

* Added the ability to send reports through an HTTP or SOCKS5 proxy.

  To enable reporting through a proxy, set `"proxyUrl": "http://192.168.1.1:3128"` within the `reporting` section of the configuration.

* Added support for transport level batching, like [apollo-link-batch-http](https://github.com/apollographql/apollo-link/tree/master/packages/apollo-link-batch-http).

  By default, query batches are fractured by the proxy and individual queries are sent to origins, in parallel.
  If your origin supports batching and you'd like to pass entire batches through, set `"supportsBatch": true` within the `origins` section of the configuration.

* *BREAKING*: Changed behaviour when the proxy receives a non-GraphQL response from an origin server.
  Previously the proxy would serve the non-GraphQL response, now it returns a valid GraphQL error indicating that the origin failed to respond.

* Added support for the `includeInResponse` query extension. This allows clients to request GraphQL response extensions be forwarded through the proxy.

  To instruct the proxy to strip extensions, set: `"extensions": { "strip": ["cacheControl", "tracing", "myAwesomeExtension"] }` within the `frontends` section of the configuration.
  By default, Apollo extensions: `cacheControl` and `tracing` are stripped.

  Stripped extensions may still be returned if the client requests them via the `includeInResponse` query extension.
  To instruct the proxy to _never_ return extensions, set `"extensions": { "blacklist": ["tracing","mySecretExtension"] }` within the `frontends` section of the configuration.
  By default, the Apollo tracing extension: `tracing` is blacklisted.

* *BREAKING*: Fixed a bug where literals in a query were ignored by query cache lookup. This change invalidates the current query cache.

* Fixed a bug where the `X-Engine-From` header was not set in non-GraphQL requests forwarded to origins. This could result in an infinite request loop in `apollo-engine-js`.

### 2017.10-431-gdc135a5d

* Fixed an issue with per-type stats reporting.

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

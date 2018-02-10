---
title: Proxy release notes
order: 20
---

### 2018.02-37-g678cbb68b – 2018-02-10

* Adds support for GZIP content encoding for Lambda origins
* Adds support for function qualifiers for Lambda origins
* Allows per-endpoint origin specification on frontends via `endpointMap`, a \<string,string\> map from endpoint path to `originName`. Users can use this field instead of `endpoints` and `originName` to route different URL paths on a frontend to serve different origins. If `endpointMap` is set, the Proxy will return a 404 error to HTTP requests sent to paths that don't match one of its keys. The proxy will also verify that only one of `endpoint` [deprecated], `endpoints`, and `endpointMap` are set.
  * For example, if you have two origins with names `[adminOrigin, userOrigin]` and want to forward requests to `/admin` and `/user` respectively, on the `Frontend` config, specify `endpointMap: {'/admin':'adminOrigin', '/user':'userOrigin'}` and do not specify `endpoint` or `endpoints`.
* Fixes a bug where all custom extensions were assumed to be maps

### 2018.02-2-g0b77ff3e3 – 2018-02-05

* Fixed a bug where `Host` header was still not forwarded to origin servers if present.
* Exposes stats field to better track engineproxy memory usage

### 2018.01-54-gce490265c - 2018-01-31

* Fixed a bug where the `Host` header was not forwarded to origin servers. If the `Host` header is present, it will also be sent in the `X-Forwarded-Host` header. Both of these header values can be overridden via the field mentioned below.
* Added the ability for users to override which headers are sent to their GraphQL origin. Users can do this by specifying the `overrideRequestHeaders` field in `origin.http` in the Engine config object. By default Engine will forward all header values it receives to the origin server. This field is only for users that want to override the default behavior.
  * For example, to override the `Host` header which may need to be done when deploying Engine inside of a PaaS (such as Heroku) follow instructions [here](setup-virtual.html).

### 2018.01-43-g1747440e6 - 2018-01-29

* Fixed an issue where Engine proxy would cache responses that set a cookie, causing cache hits to set the same cookie.
  Engine proxy now skips cache for:
    * Responses with a `Set-Cookie` header.
    * Responses with a `WWW-Authenticate` header.
    * Responses with a `Cache-Control` header value of: `no-cache` ,`no-store` or `private`.
    * Responses with an `Expires` header of `0`, or any date in the past.
* Fixed several issues with timestamps included in reports sent to engine backend.
* Added the ability to dump stacktraces of all running threads when engineproxy receives a `SIGUSR2` signal.
  When requested, traces are dumped to stderr. This should not be necessary unless requested by Apollo support.
* Added the ability to collect performance data from engine proxy using [Go pprof profiler](https://golang.org/pkg/net/http/pprof/).
  To enable the pprof server, add `"debugServer": {"port": 1234}` to your engine configuration.
  Note that the pprof server offers no security, so a firewall etc is required if running in production.
  Enabling the debug server should not be necessary unless requested by Apollo support.

### 2018.01-17-g9c203510f - 2018-01-16

* Fixed an issue where a data race could cause the proxy to crash.

### 2018.01-1-gc024df504 - 2018-01-04

* Added a flag to disable certificate validation when communicating with HTTPS origins.
  To disable certificate validation, set `disableCertificateCheck: true` within the `http` section of the origin's configuration.
  This is strongly discouraged, as it leaves Engine vulnerable to man-in-the-middle attacks. It is intended for testing only.

* Added a flag to use custom certificate authorities when communicating with HTTPS origins.
  To use custom certificate authorities, set: `trustedCertificates: /etc/ssl/cert.pem` (or another file path) within the `http` section of the origin's configuration.
  CA certificates must be PEM encoded. Multiple certificates can be included in the same file.

### 2017.12-45-g12ba029f9 - 2017-12-20

* Added support for multiple endpoints per origin through a new `endpoints` setting, deprecated the previous `endpoint` setting.
* Added a health check URL at `/.well-known/apollo/engine-health`, currently returning HTTP status 200 unconditionally.
* Fixed an issue where reports would always be sent on shut down, even when reporting was disabled.
* Fixed issues with reloading of `frontend`s, and dependencies like logging and caches.

### 2017.12-28-gcc16cbea7 - 2017-12-12

* Added a flag to disable compression when communicating with HTTP origins.
  To disable compression, set `disableCompression: true` within the `http` section of the origin's configuration.
* Exposed the maximum number of idle connections to keep open between engine an an HTTP origin.
  To tune the maximum number of idle connections, set `maxIdleConnections: 1234` within the `http` section of the origin's configuration.
  If no value is provided, the default is 100.
* Fixed an issue where Engine would return an empty query duration on internal error.
* Fixed an issue where Engine would return an empty query duration on cache hit.
* Fixed an issue where configuration reloading would not affect cache stores.
* Reduced the overhead of reporting while it is disabled.
* Added support for GraphQL `"""block strings"""`.
* *Breaking*: Added `name` field to origin configurations. Every defined origin must have a unique name (the empty string is OK).
  This only affects configurations with multiple origins, which should be rare.

### 2017.11-137-g908dbec6f - 2017-12-05

* Improved persisted query handling so that cache misses are not treated like other GraphQL errors.
* Fixed an issue where GraphQL query extensions (like `persistedQuery`) would be forwarded to the origin server. This caused issues with origins other than Apollo Server.

### 2017.11-121-g2a0310e1b - 2017-11-30

* Improved performance when reverse proxying non-GraphQL requests.
* Removed `-restart=true` flag, which spawned and managed a child proxy process. This was only used by `apollo-engine-js`.
* Added POSIX signal processing:
  * On `SIGHUP`, reload configuration. Configurations provided through `STDIN` ignore `SIGHUP`.
  * On `SIGTERM`, or `SIGINT`, attempt to send final stats and traces  before gracefully shutting down.
* Added the ability to prevent certain GraphQL variables, by name, from being forwarded to Apollo Engine servers. The proxy replaces these variables with the string `(redacted)` in traces, so their presence can be verified but the value is not transmitted.

  To blacklist GraphQL variables `password` and `secret`, add: `"privateVariables": ["password", "secret"]` within the `reporting` section of the configuration. There are no default private variables.
* Added the option to disable reporting of stats and traces to Apollo servers, so that integration tests can run without polluting production data.

 To disable reporting, add `"disabled": true` within the `reporting` section of the configuration. Reporting is enabled by default.
* Added the ability to forward log output to `STDOUT`, `STDERR`, or a file path. Previously logging was always sent to `STDERR`.

 To change log output, add `"destination": "STDOUT"` within the `logging` section of the configuration.
 Like query/request loggings, rotation of file logs is out of scope.
* Fixed an issue where `Content-Type` values with parameters (e.g. `application/json;charset=utf=8`) would bypass GraphQL instrumentation.
* Added support for the Automatic Persisted Queries protocol.

### 2017.11-84-gb299b9188 - 2017-11-20

* Fixed GraphQL parsing bugs that prevented handling requests containing list literals and object literals.
* Added the ability for the proxy to output JSON formatted logs.
* Fixed a bug with reverse proxying to HTTPS origins.

### 2017.11-59-g4ff40ec30 - 2017-11-14

* Fixed passing through custom fields on GraphQL errors.

### 2017.11-40-g9585bfc6 - 2017-11-09

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

### 2017.10-431-gdc135a5d - 2017-10-26

* Fixed an issue with per-type stats reporting.

### 2017.10-425-gdd4873ae - 2017-10-26

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

---
title: Proxy config file
order: 20
---

# Config







<a name="mdg.engine.config.proto.Config"/>

### Config
The "Config" message defines the type of Apollo Engine Proxy's JSON configuration file. The JSON file encodes one instance of this message, using [standard proto3 JSON mapping](https://developers.google.com/protocol-buffers/docs/proto3#json).

The proto3 JSON mapping is mostly straightforward, but you may want to pay attention to the rules for specifying a [Duration](https://developers.google.com/protocol-buffers/docs/proto3#json).

Note that proto3 JSON flattens "oneof" messages, such that: `oneof foobar { bool foo = 1; bool bar = 2; }` is represented as: `{ "foo": true }`.

If you are configuring the Proxy via the `apollo-engine` npm package, this JSON object is passed as the `engineConfig` parameter to the `Engine` constructor.


| Field | Type | Description |
| ----- | ---- | ----------- |
| apiKey |  string | API key for the service. Get this from your [Engine home](https://engine.apollographql.com). This field is required. |
| origins | repeated  [Config.Origin](#mdg.engine.config.proto.Config.Origin)  | Origins represent the GraphQL servers to which the Proxy will send requests. If you're using the `apollo-engine` npm package, you don't need to specify origins: the package will generate one automatically for you. |
| frontends | repeated  [Config.Frontend](#mdg.engine.config.proto.Config.Frontend)  | The list of frontends to listen to for GraphQL queries. If you're using the `apollo-engine` npm package, you don't need to specify frontends: the package will generate one automatically for you. |
| stores | repeated  [Config.Store](#mdg.engine.config.proto.Config.Store)  | The list of configured stores to cache responses to. |
| sessionAuth |   [Config.SessionAuth](#mdg.engine.config.proto.Config.SessionAuth)  | The session authorization configuration to use for per-session caching. |
| logging |   [Config.Logging](#mdg.engine.config.proto.Config.Logging)  | The logging configuration to use. |
| reporting |   [Config.Reporting](#mdg.engine.config.proto.Config.Reporting)  | The reporting configuration to use. |
| queryCache |   [Config.QueryResponseCache](#mdg.engine.config.proto.Config.QueryResponseCache)  | The query response cache configuration to use. |
| persistedQueries |   [Config.PersistedQueries](#mdg.engine.config.proto.Config.PersistedQueries)  | The persisted query configuration to use. |
| debugServer |   [Config.DebugServer](#mdg.engine.config.proto.Config.DebugServer)  | Configuration for an HTTP server which can be used to debug the Proxy. __If you enable the debug server in production, you should ensure that its port is not publicly accessible, as it provides internal information about the Proxy.__ |




<a name="mdg.engine.config.proto.Config.DebugServer"/>

### Config.DebugServer
DebugServer configures an HTTP server which can be used to debug the Proxy. If you enable the debug server in production, you should ensure that its port is not publicly accessible, as it provides internal information about the Proxy. The server includes the [Go pprof profiler](https://golang.org/pkg/net/http/pprof/). Apollo support may direct you to enable this server, and send them the files created by commands such as `wget http://127.0.0.1:4444/debug/pprof/profile`.


| Field | Type | Description |
| ----- | ---- | ----------- |
| host |  string | The address on which to listen. If left blank, this will default to "127.0.0.1"; set to "0.0.0.0" to listen on all interfaces. |
| port |  int32 | The port on which to listen. Required. |




<a name="mdg.engine.config.proto.Config.Frontend"/>

### Config.Frontend
Frontend defines a web server run by the Proxy. The Proxy will listen on each frontend for incoming GraphQL requests.


| Field | Type | Description |
| ----- | ---- | ----------- |
| host |  string | The address on which to listen. If left blank, this will default to all interfaces. |
| port |  int32 | The port on which to listen. If left blank, this will select a random available port. |
| endpoint |  string | URL path on which to listen; often "/graphql". *Deprecated:* use `endpoints`. |
| endpoints | repeated string | URL paths on which to listen; often `["/graphql"]`. |
| originName |  string | Name of origin to serve with this frontend. |
| extensions |   [Config.Frontend.Extensions](#mdg.engine.config.proto.Config.Frontend.Extensions)  | Configuration for GraphQL response extensions. |




<a name="mdg.engine.config.proto.Config.Frontend.Extensions"/>

### Config.Frontend.Extensions
Configuration for GraphQL response extensions.


| Field | Type | Description |
| ----- | ---- | ----------- |
| strip | repeated string | Extensions to strip from responses returned to clients. Clients may still request these extensions, use `blacklist` for stronger protection. If not specified, defaults to all Apollo extensions: `["cacheControl","tracing"]` |
| blacklist | repeated string | Extensions to always strip, even if the client requests them. If not specified, defaults to Apollo tracing extension: `["tracing"]` |




<a name="mdg.engine.config.proto.Config.Logging"/>

### Config.Logging
The logging configuration.


| Field | Type | Description |
| ----- | ---- | ----------- |
| level |  string | Log level for the Proxy. Defaults to "INFO". Set to "DEBUG" for more verbose logging or "ERROR" for less verbose logging. |
| format |   [Config.Logging.LogFormat](#mdg.engine.config.proto.Config.Logging.LogFormat)  | Log format for the proxy. Defaults to `TEXT`. |
| destination |  string | Path for logs. Can be a file path, `STDOUT` or `STDERR`. |
| request |   [Config.Logging.AccessLogging](#mdg.engine.config.proto.Config.Logging.AccessLogging)  | Configuration for request logging, which logs every HTTP request (including non-GraphQL). |
| query |   [Config.Logging.AccessLogging](#mdg.engine.config.proto.Config.Logging.AccessLogging)  | Configuration for query logging, which logs only GraphQL queries. |




<a name="mdg.engine.config.proto.Config.Logging.AccessLogging"/>

### Config.Logging.AccessLogging
Configuration for access logging.


| Field | Type | Description |
| ----- | ---- | ----------- |
| destination |  string | Path for JSON access logs of all proxy traffic. Can be a file path, `STDOUT` or `STDERR`. |
| requestHeaders | repeated string | Request headers to include in access logs. |
| responseHeaders | repeated string | Response headers to include in access logs. |




<a name="mdg.engine.config.proto.Config.Origin"/>

### Config.Origin
An Origin is a backend that the Proxy can send GraphQL requests to. Can use one of:

1. HTTP / HTTPS

1. AWS Lambda


| Field | Type | Description |
| ----- | ---- | ----------- |
| requestTimeout |  Duration | Amount of time to wait before timing out request to this origin. If this is left unspecified, it will default to 30 secs for HTTP or use the function's `timeout` for Lambda. |
| maxConcurrentRequests |   [uint64](#uint64)  | Maximum number of concurrent requests to the origin. All requests beyond the maximum will return 503 errors. If not specified, this will default to 9999. |
| requestType |   [Config.Protocol](#mdg.engine.config.proto.Config.Protocol)  | The type of the body of a request to this origin. If not specified, will default to JSON. |
| supportsBatch |  bool | Does this origin support batched query requests, as defined by: https://github.com/apollographql/apollo-server/blob/213acbba/docs/source/requests.md#batching |
| http |   [Config.Origin.HTTP](#mdg.engine.config.proto.Config.Origin.HTTP)  | Configuration if this is an HTTP origin. |
| lambda |   [Config.Origin.Lambda](#mdg.engine.config.proto.Config.Origin.Lambda)  | Configuration if this is a Lambda origin. |
| name |  string | The name of the origin; used in other parts of the config file to reference the origin. Empty strings are valid. If not defined, defaults to the empty string. |




<a name="mdg.engine.config.proto.Config.Origin.HTTP"/>

### Config.Origin.HTTP
Configuration for forwarding GraphQL queries to an HTTP endpoint.


| Field | Type | Description |
| ----- | ---- | ----------- |
| url |  string | The backend server's GraphQL URL. Required. |
| headerSecret |  string | If set, all requests to this origin will contain this value in the X-ENGINE-FROM header. This is intended for "sidecar" configurations where the origin proxies requests to the Proxy which then proxies back to the origin. This field is set automatically by the `apollo-engine` npm package. |
| disableCompression |  bool | If set, requests to this origin will not use compression (i.e. gzip, compress, deflate). This is usually a performance improvement if engine is running on the same server as the origin. |
| maxIdleConnections |   [uint64](#uint64)  | Maximum number of idle connections to keep open. If not specified, this will default to 100. |
| trustedCertificates |  string | File path to load trusted X509 CA certificates. This should not be required if your HTTPS origin works in modern browsers. Certificates must be PEM encoded, and multiple certificates can be concatenated into a single file. If specified, only servers with a trust chain to these certificates will be accepted. If not specified, this will default to a certificate bundle included with the proxy binary, which is extracted from Ubuntu Linux. |
| disableCertificateCheck |  bool | If set, X509 certificate validity (issuer, hostname, expiration) is not verified. This is very insecure, and should only be used for testing. |
| overrideRequestHeaders | map[string]string  | If set, requests to this origin will have these headers replaced (or added) with the given values. |




<a name="mdg.engine.config.proto.Config.Origin.Lambda"/>

### Config.Origin.Lambda
Configuration for proccessing GraphQL queries in an AWS Lambda function.


| Field | Type | Description |
| ----- | ---- | ----------- |
| functionArn |  string | The handler function's ARN (formatted as "arn:aws:lambda:REGION:ACCOUNT-ID:function:FUNCTION-NAME"). Required. |
| awsAccessKeyId |  string | The AWS access key ID for an AWS IAM user with `lambda:Invoke` permissions. If this is left unspecified, it will fall back to environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, then to EC2 instance profile. |
| awsSecretAccessKey |  string | The AWS secret access key for the AWS IAM access key ID specified in the `awsAccessKeyId` field. |




<a name="mdg.engine.config.proto.Config.PersistedQueries"/>

### Config.PersistedQueries
PersistedQueries defines behaviour of the persistent query cache.


| Field | Type | Description |
| ----- | ---- | ----------- |
| store |  string | The name of the store to use in caching queries. |
| compressionThreshold |  int64 | Minimum size in bytes to trigger compression. If not specified, defaults to 1024. Set to a negative number to disable compression. |




<a name="mdg.engine.config.proto.Config.QueryResponseCache"/>

### Config.QueryResponseCache
QueryResponseCache defines the behaviour of the query response cache.


| Field | Type | Description |
| ----- | ---- | ----------- |
| publicFullQueryStore |  string | The name of the store to use in caching full query responses containing only public/shared data. |
| privateFullQueryStore |  string | The name of the store to use in caching full query responses containing only private/per-session data. |




<a name="mdg.engine.config.proto.Config.Reporting"/>

### Config.Reporting
The reporting configuration to use. Reports about the GraphQL queries and responses will be sent approximately every 5 seconds to the `endpointUrl`.


| Field | Type | Description |
| ----- | ---- | ----------- |
| endpointUrl |  string | URL to send the reports to. By default, reports will be sent to "https://engine-report.apollodata.com". |
| maxAttempts |  int32 | Reporting is retried with exponential backoff, up to this many times. This is inclusive of the original request. Must be at least zero. |
| retryMinimum |  Duration | Minimum backoff for retries. If not specified this will default to 100ms. Must be greater than 0. |
| retryMaximum |  Duration | Maximum backoff for retries. Must be greater than or equal to `retryMinimum`. |
| debugReports |  bool | Dump reports as JSON to debug logs. This is usually only used by Apollo support. |
| hostname |  string | Override for hostname reported to backend. |
| noTraceVariables |  bool | Don't include variables in query traces. |
| privateHeaders | repeated string | Headers that should not be sent to Apollo servers. These are case-sensitive. |
| proxyUrl |  string | URL to proxy reporting requests through. `socks5://` and `http://` proxies are supported. |
| privateVariables | repeated string | Names of variables whose values should not be sent to Apollo servers. These are case-sensitive. |
| disabled |  bool | Disable sending reports to Apollo servers. |




<a name="mdg.engine.config.proto.Config.SessionAuth"/>

### Config.SessionAuth
SessionAuth describes how the Proxy identifies clients for `private` cache responses. Optionally, it can tell the Proxy how to authenticate sessions.


| Field | Type | Description |
| ----- | ---- | ----------- |
| header |  string | The header that contains an authentication token. Set either this field or "cookie". |
| cookie |  string | The cookie that contains an authentication token. Set either this field or "header". |
| tokenAuthUrl |  string | The URL to use in validating the session authentication token. The Proxy will submit an HTTP POST to this URL with a JSON body containing: `{"token": "AUTHENTICATION-TOKEN"}`. The response body should return an integer field "ttl" specifying the number of seconds to continue to consider the session to be valid, and an optional "id" field specifying a longer lived user identifier. This "id" allows the cache to span logins and/or user devices. `{"ttl": 300, "id": "user1"}` This url must respond with an HTTP 2xx for valid authentication tokens. If the returned "ttl" is 0, or no "ttl" is provided, the session is considered valid forever. If the response is an HTTP 401 or 403, or the returned "ttl" is < 0, the session is considered invalid and the request is rejected by the Proxy. |
| store |  string | The name of the store to use for caching the sessionIDs that have been verified. |




<a name="mdg.engine.config.proto.Config.Store"/>

### Config.Store
Configures a cache for GraphQL and authentication responses. Can use one of:

1. memcached

1. in-memory cache


| Field | Type | Description |
| ----- | ---- | ----------- |
| name |  string | The name of the store; used in other parts of the config file to reference the store. |
| memcache |   [Config.Store.Memcache](#mdg.engine.config.proto.Config.Store.Memcache)  | Memcache configuration |
| inMemory |   [Config.Store.InMemory](#mdg.engine.config.proto.Config.Store.InMemory)  | In-memory configuration |




<a name="mdg.engine.config.proto.Config.Store.InMemory"/>

### Config.Store.InMemory
Configures in-memory store.


| Field | Type | Description |
| ----- | ---- | ----------- |
| cacheSize |  int64 | The size of the in-memory cache in bytes. Must be greater than 512KB. If not specified, will be 5MB. Note that only values smaller than approximately 1/1024th of the in memory cache size are stored. You'll see `WARN` logs if values are overflowing this limit. __Changing this value after the proxy has launched will invalidate the current cache.__ |




<a name="mdg.engine.config.proto.Config.Store.Memcache"/>

### Config.Store.Memcache
Configures memcached store


| Field | Type | Description |
| ----- | ---- | ----------- |
| url | repeated string | The URLs for the memcached store. Currently does not support authentication. |
| timeout |  Duration | Socket read/write timeout for the store. This will default to 1s if not specified. |
| keyPrefix |  string | A prefix added to every memcached key. This allows you to share a single memcached cluster between multiple unrelated Apollo Engine Proxy configurations, or with other data. |






<a name="mdg.engine.config.proto.Config.Logging.LogFormat"/>

### Config.Logging.LogFormat
Logging formats

| Value | Description |
| ----- | ----------- |
| TEXT | Text formatting, with one log record on every line |
| JSON | JSON formatting, with one log record in each JSON object. |


<a name="mdg.engine.config.proto.Config.Protocol"/>

### Config.Protocol
Enum describing which GraphQL transport protocol is implemented by an origin. If not specified, defaults to JSON.

| Value | Description |
| ----- | ----------- |
| JSON | The standard JSON GraphQL transport is documented [here](http://graphql.org/learn/serving-over-http/#post-request) |
| CBOR | GraphQL transport over CBOR is supported by Apollo Engine Proxy but not yet documented. |







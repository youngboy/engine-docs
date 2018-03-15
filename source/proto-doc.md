---
title: Proxy configuration
order: 20
---

The Engine Proxy has many configuration options. While all you need to set to get started is your `apiKey`, many aspects of the Proxy's behavior can be fine-tuned. This page describes all the available options.

If you are using the `apollo-engine` npm package, the parameters described here are the options to the `new ApolloEngine` and `new ApolloEngineLauncher` constructors. If you are using the Docker container distribution, the parameters described here go into your `engine-config.json` file.

The `ApolloEngine` class in the `apollo-engine` npm package takes care of filling in a few important parameters for you automatically: `frontends.host`, `frontends.port`, `frontends.endpoints`, `origins.http.url`, and `origins.http.useFrontendPath`. If you are using that class, all you need is your `apiKey` to get started.

All Durations are represented as strings consisting of a number of seconds followed by the letter `s`, such as `"30s"` or `".5s"`.







<a name="mdg.engine.config.proto.Config"/>

## Top level options
These are the top-level options for the `new ApolloEngine()` or `new ApolloEngineLauncher()` constructors or for the `engine-config.json` file.


<dl>
<dt>apiKey (string)</dt>
<dd>API key for the service. Get this from [Engine](https://engine.apollographql.com) by logging in and creating a service. __Required__.</dd>
<dt>origins (array of [Origin](#mdg.engine.config.proto.Config.Origin))</dt>
<dd>Origins represent the GraphQL servers to which the Proxy will send requests. If you're using the `ApolloEngine` class from the `apollo-engine` npm package, you don't need to specify origins: the package will generate one automatically for you, or will fill in the url automatically if you have configured other origin properties. Otherwise this field is __required__.</dd>
<dt>frontends (array of [Frontend](#mdg.engine.config.proto.Config.Frontend))</dt>
<dd>Frontends represent the HTTP servers that the Proxy creates to listen for GraphQL queries. If not specified, a frontend will be created with default values.</dd>
<dt>stores (array of [Store](#mdg.engine.config.proto.Config.Store))</dt>
<dd>The list of configured stores to cache responses to.</dd>
<dt>sessionAuth ([SessionAuth](#mdg.engine.config.proto.Config.SessionAuth))</dt>
<dd>The session authorization configuration to use for per-session caching.</dd>
<dt>logging ([Logging](#mdg.engine.config.proto.Config.Logging))</dt>
<dd>The logging configuration to use.</dd>
<dt>reporting ([Reporting](#mdg.engine.config.proto.Config.Reporting))</dt>
<dd>The reporting configuration to use.</dd>
<dt>queryCache ([QueryResponseCache](#mdg.engine.config.proto.Config.QueryResponseCache))</dt>
<dd>Configuration for [caching responses to GraphQL queries](./caching.html). Only responses annotated with the cache-control extension are cached.</dd>
<dt>persistedQueries ([PersistedQueries](#mdg.engine.config.proto.Config.PersistedQueries))</dt>
<dd>Configuration for [persisted queries](./auto-persisted-queries.html).</dd>
<dt>debugServer ([DebugServer](#mdg.engine.config.proto.Config.DebugServer))</dt>
<dd>Configuration for an HTTP server which can be used to debug the Proxy. __If you enable the debug server in production, you should ensure that its port is not publicly accessible, as it provides internal information about the Proxy.__</dd>

</dl>



<a name="mdg.engine.config.proto.Config.DebugServer"/>

## DebugServer
DebugServer configures an HTTP server which can be used to debug the Proxy. If you enable the debug server in production, you should ensure that its port is not publicly accessible, as it provides internal information about the Proxy. The server includes the [Go pprof profiler](https://golang.org/pkg/net/http/pprof/). Apollo support may direct you to enable this server, and send them the files created by commands such as `wget http://127.0.0.1:4444/debug/pprof/profile`.


<dl>
<dt>host (string)</dt>
<dd>The address on which to listen. If left blank, this will default to "127.0.0.1"; set to "0.0.0.0" to listen on all interfaces.</dd>
<dt>port (int32)</dt>
<dd>The port on which to listen. __Required__.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Frontend"/>

## Frontend
Frontend defines a web server run by the Proxy. The Proxy will listen on each frontend for incoming GraphQL requests.


<dl>
<dt>host (string)</dt>
<dd>The address on which to listen. If left blank, this will default to all interfaces. (If you are using the `ApolloEngine` class from the `apollo-engine` npm module, this field defaults to the `host` option of the `engine.listen` method.)</dd>
<dt>port (int32)</dt>
<dd>The port on which to listen. If left blank, this will attempt to use the port specified in portFromEnv. If portFromEnv is left blank, this will select a random available port. (If you are using the `ApolloEngine` class from the `apollo-engine` npm module, this field defaults to the `port` option of the `engine.listen` method.)</dd>
<dt>portFromEnv (string)</dt>
<dd>The name of the environment variable to use for choosing `port`, usually "PORT". For example, when using a Docker container deployed on Heroku, you should NOT set port, and set portFromEnv to be PORT. See [the Engine docs](https://www.apollographql.com/docs/engine/setup-virtual.html) for a more detailed walkthrough on setting up Apollo Engine on Heroku and similar hosting platforms.</dd>
<dt>endpoints (array of string)</dt>
<dd>URL paths on which to listen; often `["/graphql"]`. (If you are using the `ApolloEngine` class from the `apollo-engine` npm module, this field defaults to the `graphqlPaths` option of the `engine.listen` method, which itself defaults to `["/graphql"]`.)</dd>
<dt>originName (string)</dt>
<dd>Name of origin to serve with this frontend. The Proxy will also pass any HTTP requests sent to paths not in `endpoints` to this origin. If not defined, defaults to the empty string, which is a valid origin name.</dd>
<dt>endpointMap (map&lt;string, string&gt;)</dt>
<dd>Map from URL path to origin name. Use this field instead of `endpoints` and `originName` if you want different URL paths on this frontend to serve different origins. If you use this field, the Proxy will return a 404 error to HTTP requests sent to paths that don't match one of its keys.</dd>
<dt>overrideGraphqlResponseHeaders (map&lt;string, string&gt;)</dt>
<dd>If set, HTTP responses from this frontend will have these headers added (or replaced) with the given values. These headers are only applied to GraphQL responses, not Websockets or HTTP processes outside of GraphQL paths. One use case for this is ensuring that CORS response headers are set for all GraphQL responses, including those generated by Engine itself without talking to your backend as part of the persistent query protocol. See also overrideRequestHeaders (on Origin).</dd>
<dt>extensions ([Frontend.Extensions](#mdg.engine.config.proto.Config.Frontend.Extensions))</dt>
<dd>Configuration for GraphQL response extensions.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Frontend.Extensions"/>

## Frontend.Extensions
Configuration for GraphQL response extensions.


<dl>
<dt>strip (array of string)</dt>
<dd>Extensions to strip from responses returned to clients. Clients may still request these extensions, use `blacklist` for stronger protection. If not specified, defaults to all Apollo extensions: `["cacheControl","tracing"]`</dd>
<dt>blacklist (array of string)</dt>
<dd>Extensions to always strip, even if the client requests them. If not specified, defaults to Apollo tracing extension: `["tracing"]`</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Logging"/>

## Logging
The logging configuration.


<dl>
<dt>level (string)</dt>
<dd>Log level for the Proxy. Defaults to "INFO". Set to "DEBUG" for more verbose logging or "ERROR" for less verbose logging.</dd>
<dt>format ([Logging.LogFormat](#mdg.engine.config.proto.Config.Logging.LogFormat))</dt>
<dd>Log format for the proxy. Defaults to `TEXT`.</dd>
<dt>destination (string)</dt>
<dd>Path for logs. Can be a file path, `STDOUT` or `STDERR`.</dd>
<dt>request ([Logging.AccessLogging](#mdg.engine.config.proto.Config.Logging.AccessLogging))</dt>
<dd>Configuration for request logging, which logs every HTTP request (including non-GraphQL).</dd>
<dt>query ([Logging.AccessLogging](#mdg.engine.config.proto.Config.Logging.AccessLogging))</dt>
<dd>Configuration for query logging, which logs only GraphQL queries.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Logging.AccessLogging"/>

## Logging.AccessLogging
Configuration for access logging.


<dl>
<dt>destination (string)</dt>
<dd>Path for JSON access logs of all proxy traffic. Can be a file path, `STDOUT` or `STDERR`.</dd>
<dt>requestHeaders (array of string)</dt>
<dd>Request headers to include in access logs.</dd>
<dt>responseHeaders (array of string)</dt>
<dd>Response headers to include in access logs.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Origin"/>

## Origin
An Origin is a backend that the Proxy can send GraphQL requests to. Can use one of:

1. HTTP / HTTPS

1. AWS Lambda


<dl>
<dt>requestTimeout (Duration, a string like "4.5s")</dt>
<dd>Amount of time to wait before timing out request to this origin. If this is left unspecified, it will default to 30 secs for HTTP or use the function's `timeout` for Lambda.</dd>
<dt>maxConcurrentRequests (uint64)</dt>
<dd>Maximum number of concurrent requests to the origin. All requests beyond the maximum will return 503 errors. If not specified, this will default to 9999.</dd>
<dt>requestType ([Protocol](#mdg.engine.config.proto.Config.Protocol))</dt>
<dd>The type of the body of a request to this origin. If not specified, will default to JSON.</dd>
<dt>supportsBatch (bool)</dt>
<dd>Does this origin support batched query requests, as defined by: https://github.com/apollographql/apollo-server/blob/213acbba/docs/source/requests.md#batching</dd>
<dt>http ([Origin.HTTP](#mdg.engine.config.proto.Config.Origin.HTTP))</dt>
<dd>Configuration if this is an HTTP origin. Specify at most one of this and `lambda`. If neither are specified, this defaults to an HTTP origin with no specific configuration.</dd>
<dt>lambda ([Origin.Lambda](#mdg.engine.config.proto.Config.Origin.Lambda))</dt>
<dd>Configuration if this is a Lambda origin. Specify at most one of this and `http`.</dd>
<dt>name (string)</dt>
<dd>The name of the origin; used in other parts of the config file to reference the origin. Empty strings are valid. If not defined, defaults to the empty string.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Origin.HTTP"/>

## Origin.HTTP
Configuration for forwarding GraphQL queries to an HTTP endpoint.


<dl>
<dt>url (string)</dt>
<dd>The backend server's GraphQL URL. __Required__ unless using the `ApolloEngine` class from the `apollo-engine` npm module.</dd>
<dt>useFrontendPath (bool)</dt>
<dd>If set, the path (endpoint) from the frontend's URL will be appended to this origin's URL. This allows you to set up just one Origin for an HTTP host with multiple paths serving GraphQL, by specifying multiple endpoints on the frontend. Set automatically if `ApolloEngine` is filling in the `url` field for you.</dd>
<dt>disableCompression (bool)</dt>
<dd>If set, requests to this origin will not use compression (i.e. gzip, compress, deflate). This is usually a performance improvement if engine is running on the same server as the origin.</dd>
<dt>maxIdleConnections (uint64)</dt>
<dd>Maximum number of idle connections to keep open. If not specified, this will default to 100.</dd>
<dt>trustedCertificates (string)</dt>
<dd>File path to load trusted X509 CA certificates. This should not be required if your HTTPS origin works in modern browsers. Certificates must be PEM encoded, and multiple certificates can be concatenated into a single file. If specified, only servers with a trust chain to these certificates will be accepted. If not specified, this will default to a certificate bundle included with the proxy binary, which is extracted from Ubuntu Linux.</dd>
<dt>disableCertificateCheck (bool)</dt>
<dd>If set, X509 certificate validity (issuer, hostname, expiration) is not verified. This is very insecure, and should only be used for testing.</dd>
<dt>overrideRequestHeaders (map&lt;string, string&gt;)</dt>
<dd>If set, requests to this origin will have these headers replaced (or added) with the given values. See also overrideGraphqlResponseHeaders (on Frontend).</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Origin.Lambda"/>

## Origin.Lambda
Configuration for proccessing GraphQL queries in an AWS Lambda function.


<dl>
<dt>functionArn (string)</dt>
<dd>The handler function's ARN (formatted as "arn:aws:lambda:REGION:ACCOUNT-ID:function:FUNCTION-NAME"). __Required__.</dd>
<dt>awsAccessKeyId (string)</dt>
<dd>The AWS access key ID for an AWS IAM user with `lambda:Invoke` permissions. If this is left unspecified, it will fall back to environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, then to EC2 instance profile.</dd>
<dt>awsSecretAccessKey (string)</dt>
<dd>The AWS secret access key for the AWS IAM access key ID specified in the `awsAccessKeyId` field.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.PersistedQueries"/>

## PersistedQueries
PersistedQueries defines behaviour of the persistent query cache.


<dl>
<dt>store (string)</dt>
<dd>The name of the store to use in caching queries, or "disabled" to not support automatic persisted queries.</dd>
<dt>compressionThreshold (int64)</dt>
<dd>Minimum size in bytes to trigger compression. If not specified, defaults to 1024. Set to a negative number to disable compression.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.QueryResponseCache"/>

## QueryResponseCache
QueryResponseCache defines the behaviour of the query response cache.


<dl>
<dt>publicFullQueryStore (string)</dt>
<dd>The name of the store (by default, the empty string) to use in caching full query responses containing only public/shared data, or "disabled" to not cache public full query responses. Only responses annotated with the cache-control extension are cached.</dd>
<dt>privateFullQueryStore (string)</dt>
<dd>The name of the store (by default, the empty string) to use in caching full query responses containing only private/per-session data, or "disabled" to not cache private full query responses. Only responses annotated with the cache-control extension are cached, and this is ignored unless the sessionAuth field is specified.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Reporting"/>

## Reporting
The reporting configuration to use. Reports about the GraphQL queries and responses will be sent approximately every 5 seconds to the `endpointUrl`.


<dl>
<dt>endpointUrl (string)</dt>
<dd>URL to send the reports to. By default, reports will be sent to "https://engine-report.apollodata.com".</dd>
<dt>maxAttempts (int32)</dt>
<dd>Reporting is retried with exponential backoff, up to this many times. This is inclusive of the original request. Must be at least zero.</dd>
<dt>retryMinimum (Duration, a string like "4.5s")</dt>
<dd>Minimum backoff for retries. If not specified this will default to 100ms. Must be greater than 0.</dd>
<dt>retryMaximum (Duration, a string like "4.5s")</dt>
<dd>Maximum backoff for retries. Must be greater than or equal to `retryMinimum`.</dd>
<dt>debugReports (bool)</dt>
<dd>Dump reports as JSON to debug logs. This is usually only used by Apollo support.</dd>
<dt>hostname (string)</dt>
<dd>Override for hostname reported to backend.</dd>
<dt>noTraceVariables (bool)</dt>
<dd>Don't include variables in query traces.</dd>
<dt>privateHeaders (array of string)</dt>
<dd>Headers that should not be sent to Apollo servers. These are case-sensitive.</dd>
<dt>proxyUrl (string)</dt>
<dd>URL to proxy reporting requests through. `socks5://` and `http://` proxies are supported.</dd>
<dt>privateVariables (array of string)</dt>
<dd>Names of variables whose values should not be sent to Apollo servers. These are case-sensitive.</dd>
<dt>disabled (bool)</dt>
<dd>Disable sending reports to Apollo servers.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.SessionAuth"/>

## SessionAuth
SessionAuth describes how the Proxy identifies clients for `private` cache responses. Optionally, it can tell the Proxy how to authenticate sessions.


<dl>
<dt>header (string)</dt>
<dd>The header that contains an authentication token. Set either this field or "cookie".</dd>
<dt>cookie (string)</dt>
<dd>The cookie that contains an authentication token. Set either this field or "header".</dd>
<dt>tokenAuthUrl (string)</dt>
<dd>The URL to use in validating the session authentication token. The Proxy will submit an HTTP POST to this URL with a JSON body containing: `{"token": "AUTHENTICATION-TOKEN"}`. The response body should return an integer field "ttl" specifying the number of seconds to continue to consider the session to be valid, and an optional "id" field specifying a longer lived user identifier. This "id" allows the cache to span logins and/or user devices. `{"ttl": 300, "id": "user1"}` This url must respond with an HTTP 2xx for valid authentication tokens. If the returned "ttl" is 0, or no "ttl" is provided, the session is considered valid forever. If the response is an HTTP 401 or 403, or the returned "ttl" is < 0, the session is considered invalid and the request is rejected by the Proxy.</dd>
<dt>store (string)</dt>
<dd>The name of the store (by default, the empty string) to use for caching the sessionIDs that have been verified, or "disabled" to not cache verified sessionIDs. (This field is ignored unless tokenAuthUrl is set.)</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Store"/>

## Store
Configures a cache for GraphQL and authentication responses. Can use one of:

1. memcached

1. in-memory cache

By default, there exist an in-memory store with the default size and the empty string as a name, which is used by all features that use a store unless they specify a different store name or "disabled".


<dl>
<dt>name (string)</dt>
<dd>The name of the store; used in other parts of the config file to reference the store. May be the empty string.</dd>
<dt>memcache ([Store.Memcache](#mdg.engine.config.proto.Config.Store.Memcache))</dt>
<dd>Memcache configuration. Specify at most one of this and `inMemory`.</dd>
<dt>inMemory ([Store.InMemory](#mdg.engine.config.proto.Config.Store.InMemory))</dt>
<dd>In-memory configuration. Specify at most one of this and `memcache`. If neither are specified, this defaults to an in-memory cache with no specific configuration.</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Store.InMemory"/>

## Store.InMemory
Configures in-memory store.


<dl>
<dt>cacheSize (int64)</dt>
<dd>The size of the in-memory cache in bytes. Must be greater than 512KB. If not specified, will be 50MB. Note that only values smaller than approximately 1/1024th of the in memory cache size are stored. You'll see `WARN` logs if values are overflowing this limit. __Changing this value after the proxy has launched will invalidate the current cache.__</dd>

</dl>



<a name="mdg.engine.config.proto.Config.Store.Memcache"/>

## Store.Memcache
Configures memcached store


<dl>
<dt>url (array of string)</dt>
<dd>Addresses (hostname:port) of [memcached](http://memcached.org/) servers.. Currently does not support authentication.</dd>
<dt>timeout (Duration, a string like "4.5s")</dt>
<dd>Socket read/write timeout for the store. This will default to 1s if not specified.</dd>
<dt>keyPrefix (string)</dt>
<dd>A prefix added to every memcached key. This allows you to share a single memcached cluster between multiple unrelated Apollo Engine Proxy configurations, or with other data.</dd>

</dl>








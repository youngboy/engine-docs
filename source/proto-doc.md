---
title: Proxy config file
order: 20
---

# Config

TOC

Caching Configuration 


<a name="mdg.engine.config.proto.Config"/>
<h3 id="Config" title="Config">Config</h3>
The "Config" message defines the type of Apollo Engine Proxy's JSON configuration file. The JSON file encodes one instance of this message, using [standard proto3 JSON mapping](https://developers.google.com/protocol-buffers/docs/proto3#json).

The proto3 JSON mapping is mostly straightforward, but you may want to pay attention to the rules for specifying a [Duration](https://developers.google.com/protocol-buffers/docs/proto3#json).

Note that proto3 JSON flattens "oneof" messages, such that: `oneof foobar { bool foo = 1; bool bar = 2; }` is represented as: `{ "foo": true }`.

If you are configuring the Proxy via the `apollo-engine` npm package, this JSON object is passed as the `engineConfig` parameter to the `Engine` constructor.


| Field | Type | Description |
| ----- | ---- | ----------- |
| apiKey |  string | API key for the service. Get this from your [Engine home](https://engine.apollographql.com). This field is required for all implementations. |
| origins | repeated  [Config.Origin](#mdg.engine.config.proto.Config.Origin)  | Origins represent the GraphQL servers to which the Proxy will send requests. Require for Engine running in the standalone Docker setup. If you're using the `apollo-engine` npm package, you don't need to specify origins: the package will generate one automatically for you. |
| frontends | repeated  [Config.Frontend](#mdg.engine.config.proto.Config.Frontend)  | The list of frontends to listen to for GraphQL queries. Require for Engine running in the standalone Docker setup.  If you're using the `apollo-engine` npm package, you don't need to specify frontends: the package will generate one automatically for you. |
| stores | repeated  [Config.Store](#mdg.engine.config.proto.Config.Store)  | Feature: Caching Require if caching query responses. The list of configured stores to cache responses to. |
| sessionAuth |   [Config.SessionAuth](#mdg.engine.config.proto.Config.SessionAuth)  | Feature: Caching The session authorization configuration to use for per-session caching. **Required for secure private caching** |
| logging |   [Config.Logging](#mdg.engine.config.proto.Config.Logging)  | Default: Info. The logging configuration to use. |
| reporting |   [Config.Reporting](#mdg.engine.config.proto.Config.Reporting)  | The reporting configuration to use. Optional. Recommended to keep as default or if using privateHeaders or privateVariables. |
| queryCache |   [Config.QueryCache](#mdg.engine.config.proto.Config.QueryCache)  | Feature: Caching - RequiredThe query cache configuration to use. |




<a name="mdg.engine.config.proto.Config.Frontend"/>

<h3 id="Config.Frontend" title="Config.Frontend">Config.Frontend</h3>
Frontend defines a web server run by the Proxy. The Proxy will listen on each frontend for incoming GraphQL requests.


| Field | Type | Description |
| ----- | ---- | ----------- |
| host |  string | The address on which to listen. If left blank, this will default to all interfaces. |
| port |  int32 | The port on which to listen. If left blank, this will select a random available port. |
| endpoint |  string | URL path on which to listen; often "/graphql". |




<a name="mdg.engine.config.proto.Config.Logging"/>

<h3 id="Config.Logging" title="Config.Logging">Config.Logging</h3>
The logging configuration.


| Field | Type | Description |
| ----- | ---- | ----------- |
| level |  string | Log level for the Proxy. Defaults to "INFO". Set to "DEBUG" for more verbose logging or "ERROR" for less verbose logging. |
| request |   [Config.Logging.AccessLogging](#mdg.engine.config.proto.Config.Logging.AccessLogging)  | Configuration for request logging, which logs every HTTP request (including non-GraphQL). |
| query |   [Config.Logging.AccessLogging](#mdg.engine.config.proto.Config.Logging.AccessLogging)  | Configuration for query logging, which logs only GraphQL queries. |




<a name="mdg.engine.config.proto.Config.Logging.AccessLogging"/>

<h3 id="Config.Logging.AccessLogging" title="Config.Logging.AccessLogging">Config.Logging.AccessLogging</h3>
Configuration for access logging.


| Field | Type | Description |
| ----- | ---- | ----------- |
| destination |  string | Path for JSON access logs of all proxy traffic. Can be a file path, `STDOUT` or `STDERR`. |
| requestHeaders | repeated string | Request headers to include in access logs. |
| responseHeaders | repeated string | Response headers to include in access logs. |




<a name="mdg.engine.config.proto.Config.Origin"/>

<h3 id="Config.Origin" title="Config.Origin">Config.Origin</h3>
An Origin is a backend that the Proxy can send GraphQL requests to. Can use one of:

1. HTTP / HTTPS

1. AWS Lambda


| Field | Type | Description |
| ----- | ---- | ----------- |
| requestTimeout |  Duration | Amount of time to wait before timing out request to this origin. If this is left unspecified, it will default to 30 secs for HTTP or use the function's `timeout` for Lambda. |
| maxConcurrentRequests |   [uint64](#uint64)  | Maximum number of concurrent requests to the origin. All requests beyond the maximum will return 503 errors. If not specified, this will default to 9999. |
| requestType |   [Config.Protocol](#mdg.engine.config.proto.Config.Protocol)  | The type of the body of a request to this origin. If not specified, will default to JSON. |
| http |   [Config.Origin.HTTP](#mdg.engine.config.proto.Config.Origin.HTTP)  | Configuration if this is an HTTP origin. |
| lambda |   [Config.Origin.Lambda](#mdg.engine.config.proto.Config.Origin.Lambda)  | Configuration if this is a Lambda origin. |




<a name="mdg.engine.config.proto.Config.Origin.HTTP"/>

<h3 id="Config.Origin.HTTP" title="Config.Origin.HTTP">Config.Origin.HTTP</h3>
Configuration for forwarding GraphQL queries to an HTTP endpoint.


| Field | Type | Description |
| ----- | ---- | ----------- |
| url |  string | The backend server's GraphQL URL. Required. |
| headerSecret |  string | If set, all requests to this origin will contain this value in the X-ENGINE-FROM header. This is intended for "sidecar" configurations where the origin proxies requests to the Proxy which then proxies back to the origin. This field is set automatically by the `apollo-engine` npm package. |




<a name="mdg.engine.config.proto.Config.Origin.Lambda"/>

<h3 id="Config.Config.Origin.Lambda" title="Config.Config.Origin.Lambda">Config.Origin.Lambda</h3>
Configuration for proccessing GraphQL queries in an AWS Lambda function.


| Field | Type | Description |
| ----- | ---- | ----------- |
| functionArn |  string | The handler function's ARN (formatted as "arn:aws:lambda:REGION:ACCOUNT-ID:function:FUNCTION-NAME"). Required. |
| awsAccessKeyId |  string | The AWS access key ID for an AWS IAM user with `lambda:Invoke` permissions. If this is left unspecified, it will fall back to environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, then to EC2 instance profile. |
| awsSecretAccessKey |  string | The AWS secret access key for the AWS IAM access key ID specified in the `awsAccessKeyId` field. |




<a name="mdg.engine.config.proto.Config.QueryCache"/>

<h3 id="Config.QueryResponseCache" title="Config.QueryResponseCache">Config.QueryResponseCache</h3>
QueryResponseCache defines the behaviour of the query cache.


| Field | Type | Description |
| ----- | ---- | ----------- |
| publicFullQueryStore |  string | The name of the store to use in caching full query responses containing only public/shared data. Required |
| privateFullQueryStore |  string | The name of the store to use in caching full query responses containing only private/per-session data. **Warning: If sessionAuth not set, privateFullQueryStore does not check Authentication, but will access privateFullQueryStore. If you are setting this, should set session auth or will expose data** |




<a name="mdg.engine.config.proto.Config.Reporting"/>

<h3 id="Config.Reporting" title="Config.Reporting">Config.Reporting</h3>
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
| privateHeaders | repeated string | Headers that should not be forwarded in traces. These are case-sensitive. |




<a name="mdg.engine.config.proto.Config.SessionAuth"/>

<h3 id="Config.SessionAuth" title="Config.SessionAuth">Config.SessionAuth</h3>
SessionAuth describes how the Proxy identifies clients for `private` cache responses. Optionally, it can tell the Proxy how to authenticate sessions.


| Field | Type | Description |
| ----- | ---- | ----------- |
| header |  string | The header that contains an authentication token. Set either this field or "cookie". |
| cookie |  string | The cookie that contains an authentication token. Set either this field or "header". |
| tokenAuthUrl |  string | The URL to use in validating the session authentication token. The Proxy will submit an HTTP POST to this URL with a JSON body containing: `{"token": "AUTHENTICATION-TOKEN"}`. The response body should return an integer field "ttl" specifying the number of seconds to continue to consider the session to be valid, and an optional "id" field specifying a longer lived user identifier. This "id" allows the cache to span logins and/or user devices. `{"ttl": 300, "id": "user1"}` This url must respond with an HTTP 2xx for valid authentication tokens. If the returned "ttl" is 0, or no "ttl" is provided, the session is considered valid forever. If the response is an HTTP 401 or 403, or the returned "ttl" is < 0, the session is considered invalid and the request is rejected by the Proxy. |
| store |  string | The name of the store to use for caching the sessionIDs that have been verified. |




<a name="mdg.engine.config.proto.Config.Store"/>

<h3 id="config.Store" title="Config.Store">Config.Store [Required]</h3>
Must set up persisted queries block, query cache or sessionAuth. 

Configures a cache for GraphQL and authentication responses. Can use one of:

1. memcached - For multiple instances of Engine running, this will cache to a shared cache, or if you'd like your cache to survive during Engine restarts (recommended for production environments, if able to run a memcache server) Stores values for keys in a separate server.

1. in-memory cache - In-memory cache stores data as a persisted cache within the Engine process stores values for keys in an individual Engine process. Recommended for single applications, test environments or Galaxy users.


| Field | Type | Description |
| ----- | ---- | ----------- |
| name |  string | The name of the store; used in other parts of the config file to reference the store. |
| memcache |   [Config.Store.Memcache](#mdg.engine.config.proto.Config.Store.Memcache)  | Memcache configuration Required if inmem not set |
| inMemory |   [Config.Store.InMemory](#mdg.engine.config.proto.Config.Store.InMemory)  | In-memory configuration required if memcach|




<a name="mdg.engine.config.proto.Config.Store.InMemory"/>

<h3 id="Config.Store.InMemory" title="Config.Store.InMemory">Config.Store.InMemory</h3>
Configures in-memory store.

**Required if inmem not set
| Field | Type | Description |
| ----- | ---- | ----------- |
| cacheSize |  int64 | The size of the in-memory cache in bytes. Must be greater than 512KB. If not specified, will be 5MB. Note that only values smaller than approximately 1/1024th of the in memory cache size are stored. You'll see `WARN` logs if responses are overflowing this limit. |




<a name="mdg.engine.config.proto.Config.Store.Memcache"/>

<h3 id="Config.Store.Memcache" title="Config.Store.Memcache">Config.Store.Memcache</h3>
Configures memcached store

**Required if memcache not set
| Field | Type | Description |
| ----- | ---- | ----------- |
| url | repeated string | The URLs for the memcached store. Currently does not support authentication. |
| timeout |  Duration | Socket read/write timeout for the store. This will default to 1s if not specified. |
| keyPrefix |  string | A prefix added to every memcached key. This allows you to share a single memcached cluster between multiple unrelated Apollo Engine Proxy configurations, or with other data. |






<a name="mdg.engine.config.proto.Config.Protocol"/>

<h3 id="Config.Protocol" title="Config.Protocol">Config.Protocol</h3>
Enum describing which GraphQL transport protocol is implemented by an origin. If not specified, defaults to JSON.

| Value | Description |
| ----- | ----------- |
| JSON | The standard JSON GraphQL transport is documented [here](http://graphql.org/learn/serving-over-http/#post-request) |
| CBOR | GraphQL transport over CBOR is supported by Apollo Engine Proxy but not yet documented. |







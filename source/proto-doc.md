---
title: Proxy config file
order: 20
---

# Protocol Documentation
<a name="top"/>

## Table of Contents


- Messages
  + [Config](#mdg.engine.config.proto.Config)
  + [Config.Frontend](#mdg.engine.config.proto.Config.Frontend)
  + [Config.Logging](#mdg.engine.config.proto.Config.Logging)
  + [Config.Origin](#mdg.engine.config.proto.Config.Origin)
  + [Config.Reporting](#mdg.engine.config.proto.Config.Reporting)

- Enums
  + [Config.OriginType](#mdg.engine.config.proto.Config.OriginType)
  + [Config.Protocol](#mdg.engine.config.proto.Config.Protocol)










<a name="mdg.engine.config.proto.Config"/>

### Config
The "Config" message defines the type of Apollo Engine Proxy's JSON configuration file. The JSON file encodes one instance of this message, using the standard proto3 JSON mapping: https://developers.google.com/protocol-buffers/docs/proto3#json

The proto3 JSON mapping is mostly straightforward, but you may want to pay attention to the rules for specifying a Duration.

Note that proto3 JSON flattens "oneof" messages, such that: `oneof foobar { bool foo = 1; bool bar = 2; }` is represented as: `{ "foo": true }`.

If you are configuring the Proxy via the apollo-engine npm package, this JSON object is passed as the `engineConfig` parameter to the `Engine` constructor.


| Field | Type | Description |
| ----- | ---- | ----------- |
| apiKey |  string | API key for the service. Get this from https://engine.apollographql.com This field is required. |
| origins | repeated  [Config.Origin](#mdg.engine.config.proto.Config.Origin)  | Origins represent the GraphQL servers to which the Proxy will send requests. If you're using the apollo-engine npm package, you don't need to specify origins: the package will generate one automatically for you. |
| frontends | repeated  [Config.Frontend](#mdg.engine.config.proto.Config.Frontend)  | The list of frontends to listen to for GraphQL queries. If you're using the apollo-engine npm package, you don't need to specify frontends: the package will generate one automatically for you. |
| logcfg |   [Config.Logging](#mdg.engine.config.proto.Config.Logging)  | The logging configuration to use. |
| reporting |   [Config.Reporting](#mdg.engine.config.proto.Config.Reporting)  | The reporting configuration to use. |




<a name="mdg.engine.config.proto.Config.Frontend"/>

### Config.Frontend
Frontend defines a web server run by the Proxy. The Proxy will listen on each frontend for incoming GraphQL requests.


| Field | Type | Description |
| ----- | ---- | ----------- |
| host |  string | The address on which to listen. If left blank, this will default to all interfaces. |
| port |  int32 | The port on which to listen. If left blank, this will select a random available port. |
| endpoint |  string | URL path on which to listen; often "/graphql". |




<a name="mdg.engine.config.proto.Config.Logging"/>

### Config.Logging
The logging configuration.


| Field | Type | Description |
| ----- | ---- | ----------- |
| level |  string | Log level for the Proxy. Defaults to "INFO". Set to "DEBUG" for more verbose logging or "ERROR" for less verbose logging. |




<a name="mdg.engine.config.proto.Config.Origin"/>

### Config.Origin
An Origin is a backend GraphQL server that the Proxy can send GraphQL requests to.


| Field | Type | Description |
| ----- | ---- | ----------- |
| url |  string | For HTTP origins, this is the backend server's GraphQL URL. For Lambda origins, this is the AWS arn (formatted as "arn:aws:lambda:REGION:ACCOUNT-ID:function:FUNCTION-NAME"). Required. |
| requestTimeout |  Duration | Amount of time to wait before timing out request to this origin. If this is left unspecified, it will default to 30 secs for HTTP or use the function's timeout for Lambda. |
| maxConcurrentRequests |  int32 | Maximum number of concurrent requests to the origin. All requests beyond the maximum will return 503 errors. If not specified, this will default to 9999. |
| requestType |   [Config.Protocol](#mdg.engine.config.proto.Config.Protocol)  | The type of the body of a request to this origin. If not specified, will default to JSON. |
| originType |   [Config.OriginType](#mdg.engine.config.proto.Config.OriginType)  | The type of origin in question. If not specified, will default to HTTP. |
| awsAccessKeyId |  string | For Lambda origins: The AWS access key ID for an AWS IAM user with `lambda:Invoke` permissions. If this is left unspecified, it will fall back to environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, then to EC2 instance profile. |
| awsSecretAccessKey |  string | For Lambda origins: The AWS secret access key for the AWS IAM access key ID specified in the ID field. |
| headerSecret |  string | For HTTP origins: If set, all requests to this origin will contain this value in the X-ENGINE-FROM header.	 This is intended for "sidecar" configurations where the origin proxies requests to the Proxy which then proxies back to the origin. This field is set automatically by the apollo-engine npm package. |




<a name="mdg.engine.config.proto.Config.Reporting"/>

### Config.Reporting
The reporting configuration to use. Reports about the GraphQL queries and responses will be sent approximately every minute to the endpointUrl.


| Field | Type | Description |
| ----- | ---- | ----------- |
| endpointUrl |  string | URL to send the reports to. By default, reports will be sent to "https://engine-report.apollodata.com". |
| maxAttempts |  int32 | Reporting is retried with exponential backoff, up to this many times. This is inclusive of the original request. Must be at least zero. |
| retryMinimum |  Duration | Minimum backoff for retries. If not specified this will default to 100ms. Must be greater than 0. |
| retryMaximum |  Duration | Maximum backoff for retries. Must be greater than or equal to `retryMinimum`. |
| debugReports |  bool | Dump reports as JSON to debug logs. |
| hostname |  string | Override for hostname reported to backend. |
| noTraceVariables |  bool | Don't include variables in query traces. |
| privateHeaders | repeated string | Headers that should not be forwarded in traces. These are case-sensitive. |






<a name="mdg.engine.config.proto.Config.OriginType"/>

### Config.OriginType
The OriginType enum defines a kind of GraphQL backend. If not specified, defaults to HTTP.

| Value | Description |
| ----- | ----------- |
| HTTP | A server which provides GraphQL over standard HTTP. |
| Lambda | A server which provides GraphQL by invoking an AWS Lambda function. |


<a name="mdg.engine.config.proto.Config.Protocol"/>

### Config.Protocol
Enum describing which GraphQL transport protocol is implemented by an origin. If not specified, defaults to JSON.

| Value | Description |
| ----- | ----------- |
| JSON | The standard JSON GraphQL transport is documented at http://graphql.org/learn/serving-over-http/#post-request |
| CBOR | GraphQL transport over CBOR is supported by Apollo Engine Proxy but not yet documented. |







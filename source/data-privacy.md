---
title: Data Privacy
description: Learn about what data Engine collects and how you can configure Engine to ignore PII and other sensitive information.
---

This page will walk you through what information Engine sees about your GraphQL service's request, what Engine's default behaviour to handle request data is, and how you can configure Engine to the level of data privacy your team needs.

<h2 id="architecture">Engine Architecture</h2>

Engine is composed of two components:

1. The proxy that sits between your GraphQL service and your clients that you run on-prem.
1. The Engine cloud service that your proxy reports performance metrics to.

Setting up Engine in your app is all about getting the proxy set up in front of your GraphQL server and configuring it to behave the way you want it to. You install and run the engine proxy in your own environment on-prem, either as a [sidecar next to your Node server](./setup-node.html) or as a [separately hosted process](./setup-standalone.html) that you route your client requests through. As your clients make requests to your server, the proxy reads response extension data to make caching decisions and aggregates tracing and error information into reports to send to the Engine cloud service.

<h2 id="data-collection">Data collection</h2>

While the Engine proxy sees your client request data and service response data, it only collects and forwards data that goes into the reports you see in the Engine dashboards. All information that is sent from your on-prem proxy to the out-of-band Engine cloud service is configurable and can be turned off through config options. Data is aggregated and sent approximately every 5 seconds.

<h3 id="request">Request</h3>

In this section, we'll go over which parts of your GraphQL HTTP requests are collected by Engine.

#### Variables

Engine reports the query variables for every trace sample it stores. You can tell Engine to ignore private variables using the [`privateVariables`](./proxy-config.html#Reporting) configuration option in the proxy or you can prevent all variables from being reported with [`noTraceVariables`](./proxy-config.html#Reporting).

<h4 id="http-headers">Authorization & Cookie HTTP Headers</h4>

We'll **never** collect your application's `Authorization`, `Cookie`, or `Set-Cookie` headers. Engine collects all other headers for every trace sample it stores.

If you perform authorization in another header (like `X-My-API-Key`), be sure to add this to `privateHeaders` in your [`reporting` config object](./proxy-config.html#mdg.engine.config.proto.Config.Reporting). Note that unlike headers in general, this configuration option *is* case-sensitive.

<h3 id="response">Response</h3>

Let's walk through Engine's default behavior for reporting on fields in a typical GraphQL response:

```
// GraphQL Response
{
  "data": { ... },          // Contents of this field are never sent to the engine cloud service.
  "errors": [ ... ],        // Engine uses this field to report on errors in your service.
  "extensions": {
    "tracing": { ... },     // Engine uses this field for service performance reporting.
    "cacheControl": { ... } // Engine uses this field to determine cache policies.
  }
} 
```

#### `response.data`

The Engine proxy will never send the contents of this to the Engine cloud service; the responses from your GraphQL service stay on-prem.

If you've configured Engine caching and Engine determines that a response it sees is cacheable, then the response will be stored in your [Engine cache](./caching.html#config.stores) (either in memory or as an external memcache you configure).

#### `response.errors`

If the proxy sees a response with an `"errors"` field, it will read the `message` and `locations` fields if they exist and report them to the Engine cloud service.

You can disable reporting errors to the out-of-band Engine cloud service with the [`noTraceErrors`](./proxy-config.html#Reporting) option.

<h3 id="disable-reporting" title="Disable Reporting">Disable Reporting</h3>

We've added the option to disable reporting of stats and traces to Apollo servers so that integration tests can run without polluting production data.

To disable all reporting, use the [`disabled`](./proxy-config.html#Reporting) option.

<h2 id="policies" title="Policies and Agreements">Policies and Agreements</h2>

To learn about other ways that we protect your data, please read over our Terms of Service and Privacy Policy.

[Terms of Service](https://www.apollographql.com/policies/terms)
[Privacy Policy](https://www.apollographql.com/policies/privacy)

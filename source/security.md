---
title: Security Features
description: Learn how Apollo Engine works to secure your data
---

Apollo Engine is a proxy service that acts as a GraphQL gateway, capable of caching your GraphQL queries, tracing performance metrics, tracking errors, and more. Because Engine reports and aggregates your GraphQL query metrics and errors, there are a few actions you can take to prevent variable names or reports to be sent to the [Engine Service](https://engine.apollographql.com).

<h3 id="variable-blacklisting" title="Variable Blacklisting">Variable Blacklisting</h3>

Engine allows you to specify configuration that prevents certain GraphQL variables from being forwarded to Apollo Engine servers and reported to the [Engine Service UI](https://engine.apollographql.com). The proxy replaces these variables with the string (redacted) in traces, so their presence can be verified but the value is not transmitted.

To blacklist GraphQL variables password and secret, add: "privateVariables": ["password", "secret"] within the reporting section of the configuration. There are no default private variables.

To log the reports sent to the Engine Service UI as JSON, set [`reporting.debugReports`](https://www.apollographql.com/docs/engine/proto-doc.html) to true.

<h3 id="disable-reporting" title="Disable Reporting">Disable Reporting</h3>

We've also added the option to disable reporting of stats and traces to Apollo servers, so that integration tests can run without polluting production data.

To disable reporting, add "disabled": true within the reporting section of the configuration. Reporting is enabled by default.

<h3 id="http-headers">Authorization & Cookie HTTP Headers</h3>

We'll **never** collect your applications `Authorization` headers, `Cookie` or `Set-Cookie` headers. 

<h4 id="custom-auth-headers" title="Configure Custom Auth Headers">How to Configure Custom Authorization Headers</h4>

If you perform authorization in another header (like `X-My-API-Key`), be sure to add this to `privateHeaders`.

<h3 id="policies" title="Policies and Agreements">Policies and Agreements</h3>

To learn about other ways that we protect your data, please read over our Terms of Service and Privacy Policy.

[Terms of Service](https://www.apollographql.com/policies/terms)
[Privacy Policy](https://www.apollographql.com/policies/privacy)
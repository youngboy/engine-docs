---
title: Apollo Engine Documentation
sidebar_title: Overview
description: Learn how Apollo Engine works, and how to set it up for performance monitoring, error tracking, and more.
---

[Apollo Engine](https://www.apollographql.com/engine/) is a GraphQL gateway that helps you take GraphQL services into production with confidence.

Engine sits between your clients and your GraphQL server, delivering essential capabilities like query caching, error tracking, automatic persisted queries, and execution tracing. It works with any spec-compliant GraphQL server, including Apollo Server, GraphQL-Ruby, Sangria, and Absinthe.

To get started, [go to the Apollo Engine app and sign in](https://engine.apollographql.com/).

For details about how to integrate Engine into your GraphQL server, select your server language and environment in the sidebar on the left. To find out more about what it does and how it works, read on!

<h2 id="features">Features</h2>

Engine is designed to be your one-stop-shop for GraphQL-specific infrastructure. It provides the following features today:

1. [Performance tracing](./performance.html)
1. [Error tracking](./error-tracking.html)
1. [Response caching](./caching.html)
1. [Automatic persisted queries](./auto-persisted-queries.html)
1. Schema analytics

To learn more about these features help you work with GraphQL, check out the [Engine landing page](https://www.apollographql.com/engine/).

<h2 id="components">Engine components</h2>

Engine consists of two parts:

1. **Engine Proxy:** A thin, fast proxy that is deployed into your own cloud or datacenter, sitting in the request path right next to your GraphQL services.
2. **Engine service:** This runs in our cloud, is centrally managed, monitored, and automatically scaled for you. This is the part that displays the management UI for Engine and stores aggregated data about performance, errors, cache hits, and more.

We've designed it in this way so that your application data never leaves your infrastructure. It also allows us to build features that process your GraphQL requests and responses, such as response caching, automatic persisted queries, and more.

<h3 id="engine-proxy">Engine Proxy</h3>

Engine uses a proxy component written in Go that runs inside your infrastructure. This component is designed to allow all of your requests and responses to pass through, while doing things like extracting trace data, caching results, and more. It's designed to handle large volumes of traffic comfortably, without overloading. The Engine proxy component doesn't rely on accessing the Engine cloud service to function.

There are two options for running the Engine proxy:

- **Standalone container**: To have full control over the proxy component, you can easily deploy it as a standalone docker container the same way you deploy other parts of your application.
- **Sidecar package for Node**: If you're using Node, you can use the Engine Proxy as a [sidecar package](/docs/engine/#sidecar-package) that runs as a child process of your server. This option is simpler to set up and doesn't involve a container.

<h3 id="engine-service">Engine Service</h3>

The Engine Service is the part of Engine that runs in our cloud. It consumes reports from the proxy, aggregates data, and displays a management UI so you know what's going on inside your GraphQL infrastructure. This is the application that you sign into at <https://engine.apollographql.com/>.

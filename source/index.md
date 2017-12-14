---
title: Engine Overview
order: 0
---

<h2 id="what-is-apollo-engine" title="What is Apollo Engine">What is Apollo Engine?</h2>

[Apollo Engine](https://www.apollographql.com/engine/) is turnkey infrastructure that helps you take GraphQL services into production with confidence. The successor to Apollo Optics, Engine sits between your clients and your GraphQL server, delivering essential capabilities like query caching, error tracking, automatic persisted queries, and execution tracing on top of any spec-compliant GraphQL server including Apollo Server, GraphQL-Ruby, Sangria, and Absinthe.

Engine consists of a thin proxy that is deployed into your own cloud or datacenter, sitting in the request path right next to your GraphQL services. The Engine service runs in our cloud, is centrally managed, monitored, and automatically scaled for you.

Enabling your app with Engine is twofold.
1. Enable [Apollo Tracing](#apollo-tracing) for your GraphQL server.
2. Host [Engine's proxy](#engine-proxy) in front of your GraphQL server.

<h2 id="apollo-tracing">Apollo Tracing</h2>

[Apollo Tracing](https://github.com/apollographql/apollo-tracing) is a GraphQL extension to expose performance tracing data as part of your GraphQL responses. Engine relies on receiving data in the Apollo Tracing format to create its performance telemetry reports.

There are currently implementations that allow you to use the Apollo Tracing format with the following GraphQL servers:

1. **Node** with [Apollo Server](https://www.apollographql.com/docs/apollo-server/): [apollo-tracing-js](https://github.com/apollographql/apollo-tracing-js). supports Express, Hapi, Koa, Restify, and Lambda.
2. **Ruby** with [GraphQL-Ruby](http://graphql-ruby.org/): [apollo-tracing-ruby](https://github.com/uniiverse/apollo-tracing-ruby)
3. **Java** with [GraphQL-Java](https://github.com/graphql-java/graphql-java): Built in! [Read the docs](http://graphql-java.readthedocs.io/en/v6/instrumentation.html#apollo-tracing-instrumentation)
4. **Scala** with [Sangria](https://github.com/sangria-graphql/sangria): [Use this snippet](https://gist.github.com/OlegIlyenko/124b55e58609ad45fcec276f15158d16)
5. **Elixir** with [Absinthe](https://github.com/absinthe-graphql/absinthe): [apollo-tracing-elixir](https://github.com/sikanhe/apollo-tracing-elixir)

Using a different server? [Let us know](mailto:support@apollodata.com) – the development of our tracing agents is community driven and we would love to start a conversation with you!

<h2 id="engine-proxy">Engine Proxy</h2>

There are two options for deploying the Engine proxy. You can deploy it either as a [sidecar package](/docs/engine/#sidecar-package) that runs next to your server, or as a [standalone docker container](/docs/engine/#standalone-docker-container).

_Server compatibility chart:_

| Server  | Sidecar Package  | Docker Container |
| :------ | :------------------- | :--------------------------- |
| Apollo Server | Supported | Supported |
| GraphQL-Ruby | _Not available_ | Supported |
| GraphQL-Java | _Not available_ | Supported |
| Sangria | _Not available_ | Supported |
| Absinthe | _Not available_ | Supported |

<h3 id="sidecar-package" title="Sidecar Package">Option 1: Install using a Sidecar Package</h3>

**Available for:** Node servers (Apollo Server and Express-GraphQL)

The sidecar is a language-specific package that you add to your server. It runs a proxy next to your server in the same container as your server.

The sidecar package includes a pre-built copy of the Engine proxy binary. It spawns an Engine process side-by-side with the your GraphQL server's process and incoming GraphQL operations get routed through the proxy and then into your server.

If you are already using [Apollo Optics](https://www.apollodata.com/optics/), the sidecar does not have to replace the [Optics agent](https://github.com/apollographql/optics-agent-js) (which is a tracing agent). You can run the Engine proxy and an Apollo Tracing package in conjunction with the Optics agent, which uses a slightly different data format to report trace information.

_Choose this configuration in environments where you don't want to orchestrate and manage the proxy process separate from your GraphQL server._

<h3 id="standalone-docker-container" title="Docker Container">Option 2: Standalone Docker Container</h3>

**Available for:** All servers (Node, Java, Scala, Ruby, and Elixir)

This is a Docker container that contains the Engine proxy process. You would use this option if you want to deploy and manage the Engine proxy as a separate process from your GraphQL server.

The Engine proxy should be configured to sit in front of your GraphQL server, so that requests to your app can pass through it.

_Choose this configuration in environments where you want to control the orchestration and management of the Engine proxy directly. If your GraphQL server is running on Lambda, you'll need to use this option._


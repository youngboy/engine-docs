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

<h2 id="engine-proxy">Engine Proxy</h2>

Engine uses a proxy component written in Go that you run inside your infrastructure. This component is designed to allow all of your requests and responses to pass through, while doing things like extracting trace data, caching results, and more. It's designed to handle large volumes of traffic, and dosen't rely on the Engine cloud service to function - this is what we call our "hybrid prem" architecture, which ensures your application data never leaves your infrastructure.

There are two options for running the Engine proxy:

- **Sidecar package**: If you're using Node and looking for a quick way to get started, you can use it as a [sidecar package](/docs/engine/#sidecar-package) that runs in the same process as your server.
- **Standalone container**: To have more control over scaling the proxy component, or to use servers other than Node, you'll want to deploy the proxy as a [standalone docker container](/docs/engine/#standalone-docker-container).

<h2 id="sidecar-package">Sidecar for Node servers</h2>



The sidecar is a language-specific package that you add to your server. It runs a proxy next to your server in the same container as your server.

The sidecar package includes a pre-built copy of the Engine proxy binary. It spawns an Engine process side-by-side with the your GraphQL server's process and incoming GraphQL operations get routed through the proxy and then into your server.

If you are already using [Apollo Optics](https://www.apollodata.com/optics/), the sidecar does not have to replace the [Optics agent](https://github.com/apollographql/optics-agent-js) (which is a tracing agent). You can run the Engine proxy and an Apollo Tracing package in conjunction with the Optics agent, which uses a slightly different data format to report trace information.

_Choose this configuration in environments where you don't want to orchestrate and manage the proxy process separate from your GraphQL server._

<h2 id="standalone-docker-container">Custom Docker deployment</h2>

**Available for:** All servers (Node, Java, Scala, Ruby, and Elixir)

This is a Docker container that contains the Engine proxy process. You would use this option if you want to deploy and manage the Engine proxy as a separate process from your GraphQL server.

The Engine proxy should be configured to sit in front of your GraphQL server, so that requests to your app can pass through it.

_Choose this configuration in environments where you want to control the orchestration and management of the Engine proxy directly. If your GraphQL server is running on Lambda, you'll need to use this option._


---
title: Engine Overview
order: 0
---

## What is Apollo Engine?

Apollo Engine is an in-query-path tool that helps you understand what's happening inside your GraphQL layer and perform performance optimizations without changing any of your server's code.

Engine runs as a proxy layer in front of your GraphQL server, providing server performance telemetry (the same functionality as [Apollo Optics](https://www.apollodata.com/optics/)) and is able to cache entire query responses from your server.

Instrumenting your app with Engine is twofold.
1. Add an [Apollo Tracing package](#Apollo-Tracing-Package) (language-specific) to your GraphQL server.
2. Host [Engine's proxy](#Engine-Proxy) in front of your GraphQL server.

Apollo Engine is in Preview.

## Apollo Tracing Package

[Apollo Tracing](https://github.com/apollographql/apollo-tracing) is a GraphQL response format extension to expose trace data for GraphQL requests.

Apollo Tracing packages are language-specific packages that, once installed on a server, augment that server's GraphQL responses with the specified trace data format.

Engine relies on receiving data in the Apollo Tracing format to create its performance telemetry reports. There are currently tracing packages using the Apollo Tracing format for the following GraphQL servers:
1. **Node:** [Apollo Server](https://github.com/apollographql/apollo-server) (Express, Hapi, Koa, Restify, and Lambda); [Express-GraphQL](https://github.com/graphql/express-graphql)
2. **Ruby:** [GraphQL-Ruby](https://github.com/rmosolgo/graphql-ruby)
3. **Java:** [GraphQL-Java](https://github.com/graphql-java/graphql-java)
4. **Scala:** [Sangria](https://github.com/sangria-graphql/sangria)
5. **Elixir:** [Absinthe](https://github.com/absinthe-graphql/absinthe)

Using a different server? [Let us know](mailto:support@apollodata.com) – the development of our tracing agents is community driven and we would love to start a conversation with you!

## Engine Proxy

There are two options for deploying the Engine proxy. You can deploy it either as a [sidecar package](/#Option-1-Sidecar-Package) that runs next to your server, or as a [standalone docker container](/#Option-2-Standalone-Docker-Container).

_Server compatibility chart:_

| Server  | Sidecar Package  | Docker Container |
| :------ | :------------------- | :--------------------------- |
| Apollo Server and Express-GraphQL | Supported | Supported | 
| GraphQL-Ruby | _Not available_ | Supported |
| GraphQL-Java | _Not available_ | Supported |
| Sangria | _Not available_ | Supported |
| Absinthe | _Not available_ | Supported |

### [Option 1] Sidecar Package

**Available for:** Node servers (Apollo Server and Express-GraphQL)

The sidecar is a language-specific package that you add to your server. It runs a proxy next to your server in the same container as your server.

The sidecar package includes a pre-built copy of the Engine proxy binary. It spawns an Engine process side-by-side with the your GraphQL server's process and incoming GraphQL operations get routed through the proxy and then into your server.

If you are already using [Apollo Optics](https://www.apollodata.com/optics/), the sidecar does not have to replace the [Optics agent](https://github.com/apollographql/optics-agent-js) (which is a tracing agent). You can run the Engine proxy and an Apollo Tracing package in conjunction with the Optics agent, which uses a slightly different data format to report trace information.

_Choose this configuration in environments where you don't want to orchestrate and manage the proxy process separate from your GraphQL server._

### [Option 2] Standalone Docker Container

**Available for:** All servers (Node, Java, Scala, Ruby, and Elixir)

This is a Docker container that contains the Engine proxy process. You would use this option if you want to deploy and manage the Engine proxy as a separate process from your GraphQL server.

The Engine proxy should be configured to sit in front of your GraphQL server, so that requests to your app can pass through it.

_Choose this configuration in environments where you want to control the orchestration and management of the Engine proxy directly. If your GraphQL server is running on Lambda, you'll need to use this option._


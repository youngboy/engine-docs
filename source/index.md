---
title: Engine Overview
order: 0
---

## What is Apollo Engine?

Apollo Engine is an in-query-path tool that helps you understand what's happening in your GraphQL layer and perform performance optimizations without changing any code. Engine runs as a proxy layer in front of your GraphQL server, providing server performance telemetry (the same functionality as [Apollo Optics](https://www.apollodata.com/optics/)) and is able to cache entire query responses from your server.

**Engine is in private Early Access.** Please email [rohit@apollodata.com](mailto:rohit@apollodata.com) with requests to join the Early Access program.

## Supported Servers

Engine currently supports the following GraphQL Servers:
1. **Node:** [Apollo Server](https://github.com/apollographql/apollo-server) (Express, Hapi, Koa, Restify, and Lambda); [Express-GraphQL](https://github.com/graphql/express-graphql)
2. **Ruby:** [GraphQL-Ruby](https://github.com/rmosolgo/graphql-ruby)
3. **Java:** [GraphQL-Java](https://github.com/graphql-java/graphql-javas)
4. **Scala:** [Sangria](https://github.com/sangria-graphql/sangria)
5. **Elixir:** [Absinthe](https://github.com/absinthe-graphql/absinthe)

## Deployment Options

There are two options for deploying the Engine proxy. You can deploy it either as a **side-loader package** that runs next to your server, or as a **standalone docker container**.

_Server compatibility chart:_

| Server  | Side-Loader Package  | Docker Container |
| :------ | :------------------- | :--------------------------- |
| Apollo Server and Express-GraphQL | Supported | Supported | 
| GraphQL-Ruby | _Not available_ | Supported |
| GraphQL-Java | _Not available_ | Supported |
| Sangria | _Not available_ | Supported |
| Absinthe | _Not available_ | Supported |

### Side-Loader Package

**Available for:** Node servers

The side-loader is a language-specific package that you add to your server. It runs a proxy next to your server in the same container as your server. If you are already using [Apollo Optics](https://www.apollodata.com/optics/), the side-loader replaces the [Optics agent](https://github.com/apollographql/optics-agent-js).

The side-loader package includes a pre-built copy of the Engine proxy binary. It spawns an Engine process side-by-side with the GraphQL server process and incoming GraphQL operations get routed through the proxy and then into your server.

Choose the side-loader package in environments where you don't want to orchestrate and manage the proxy process separate from your GraphQL server.

### Standalone Docker Container

**Available for:** All servers (Node, Java, Scala, Ruby, and Elixir)

This is a Docker container that contains the Engine proxy process. You would use this option if you want to deploy and manage the Engine proxy as a separate process from your GraphQL server. Choose the standalone configuration in environments where you want to control the orchestration and management of the Engine proxy directly.

If your GraphQL server is running on Lambda, you'll need to use this option.

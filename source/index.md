---
title: Engine
order: 0
---

## What is Apollo Engine?

Engine runs as a proxy layer in front of GraphQL, provides performance telemetry (Optics functionality) and caches whole responses.

Engine is in private Early Access. Please email rohit@apollodata.com with requests to join the Early Access program.

Engine currently supports the following GraphQL Servers:

- Apollo Server - Express, Hapi, Koa, Restify and Lambda
- Express-GraphQL
- GraphQL-Ruby
- GraphQL-Java
- Sangria
- Elixir

## Deployment Configuration Options

The Engine Proxy can be deployed through either of two deployment options: **Side-loader package** or **Standalone Docker container**.

### Side-loader package

Available for: Node servers

This is a language-specific package that you add to your server - for Node, it replaces the https://github.com/apollographql/optics-agent-js agent. This package includes a pre-built copy of the Engine proxy. It spawns an Engine process side-by-side with the Graphql server process. Incoming graphql operations get routed through the proxy and then into your server.

Choose the side-loader package in environments where you don't want to orchestrate and manage the proxy process separately.

### Standalone Docker container

Available for: Node, Java, Scala and Elixir servers

This is a Docker container that contains the Engine proxy process. Use this to deploy and manage the Proxy as a separate process.

Choose the standalone configuration in environments where you want to control the orchestration and management directly. You'll need to use the standalone configuration for GraphQL server on Lambda support.
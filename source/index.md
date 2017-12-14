---
title: Engine Overview
order: 0
---

[Apollo Engine](https://www.apollographql.com/engine/) is turnkey infrastructure that helps you take GraphQL services into production with confidence. The successor to Apollo Optics, Engine sits between your clients and your GraphQL server, delivering essential capabilities like query caching, error tracking, and execution tracing.

Engine consists of a thin proxy that is deployed into your own cloud or datacenter, sitting in the request path right next to your GraphQL services. The Engine service runs in our cloud, is centrally managed, monitored, and automatically scaled for you. 

<h2 id="supported-graphql-servers" title="Supported GraphQL Servers">Supported GraphQL Servers</h2>

    Node: Apollo Server (Express, Hapi, Koa, Restify, and Lambda)
    Ruby: GraphQL-Ruby
    Java: GraphQL-Java
    Scala: Sangria
    Elixir: Absinthe

<h2 id="community" title="Community">Community</h2>

<h3 id="engine-and-apollo-tracing" title="Apollo Tracing"> GraphQL Request Performance: Apollo Tracing</h3>

Apollo Tracing augments your server’s GraphQL responses with the specified trace data format.

Engine relies on receiving data in the Apollo Tracing format to create its performance telemetry reports. 

Using a different server than the listed [Supported GraphQL Servers]/docs/engine/#? Let us know – the development of our tracing agents is community driven and we would love to start a conversation with you!



Enabling your app with Engine is twofold.
1. Enable [Apollo Tracing](#apollo-tracing) for your GraphQL server.
2. Host [Engine's proxy](#engine-proxy) in front of your GraphQL server.
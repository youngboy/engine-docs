---
title: Apollo Tracing
description: What is Apollo Tracing, and how to use it with your server.
---

[Apollo Tracing](https://github.com/apollographql/apollo-tracing) is a GraphQL extension to expose performance tracing data as part of your GraphQL responses. Engine relies on receiving data in the Apollo Tracing format to create its performance telemetry reports.

There are currently implementations that allow you to use the Apollo Tracing format with the following GraphQL servers:

1. **Node** with [Apollo Server](https://www.apollographql.com/docs/apollo-server/): [apollo-tracing-js](https://github.com/apollographql/apollo-tracing-js). supports Express, Hapi, Koa, Restify, and Lambda.
2. **Ruby** with [GraphQL-Ruby](http://graphql-ruby.org/): [apollo-tracing-ruby](https://github.com/uniiverse/apollo-tracing-ruby)
3. **Java** with [GraphQL-Java](https://github.com/graphql-java/graphql-java): Built in! [Read the docs](http://graphql-java.readthedocs.io/en/v6/instrumentation.html#apollo-tracing-instrumentation)
4. **Scala** with [Sangria](https://github.com/sangria-graphql/sangria): [Use this snippet](https://gist.github.com/OlegIlyenko/124b55e58609ad45fcec276f15158d16)
5. **Elixir** with [Absinthe](https://github.com/absinthe-graphql/absinthe): [apollo-tracing-elixir](https://github.com/sikanhe/apollo-tracing-elixir)

Using a different server? [Let us know](mailto:support@apollodata.com) – the development of our tracing agents is community driven and we would love to start a conversation with you!

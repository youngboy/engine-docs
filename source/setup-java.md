---
title: Setup for Java Servers
order: 2
---

**Supported Java servers:** [GraphQL-Java](https://github.com/graphql-java/graphql-javas)

To get started with Engine, take the following steps:

1. Configure and deploy the Engine proxy docker container
2. Install Apollo tracing for your GraphQL server
3. Deploy your server - you're all set up!

## 1. Install the Proxy

At this time, the only available option for running Engine with Java is to use the standalone docker proxy.

Skip to [the instructions](/standalone-proxy.html) for installing the standalone docker container, which are the same regardless of which language your server is written in.

## 2. Instrument Apollo Tracing

Install Apollo tracing in your GraphQL server to enable Engine to receive performance traces for your GraphQL requests.

Use the Java tracing package, with instructions here: https://github.com/graphql-java/graphql-java/pull/577

## 3. View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!

---
title: Setup for Scala Servers
order: 2
---

**Supported Scala servers:** [Sangria](https://github.com/sangria-graphql/sangria)

To get started with Engine, take the following steps:

1. Configure and deploy the Engine proxy docker container
2. Install Apollo tracing for your GraphQL server
3. Deploy your server - you're all set up!

## 1. Install the Proxy

At this time, the only available option for running Engine with Scala is to use the standalone docker proxy.

Skip to [the instructions](/standalone-proxy.html) for installing the standalone docker container, which are the same regardless of which language your server is written in.

## 2. Instrument Apollo Tracing

Install Apollo tracing in your GraphQL server to enable Engine to receive performance traces for your GraphQL requests.

Use the Scala tracing package, with instructions here: https://gist.github.com/OlegIlyenko/124b55e58609ad45fcec276f15158d16

## 3. View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!

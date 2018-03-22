---
title: Schema Analytics
description: Understand the performance usage of your GraphQL Schema
---

Engine provides a view into your GraphQL schema at two levels of granularity: full service and per operation. These two views are useful for determining your service and operations behavior across queries per field. With cross-query schema knowledge, you are better able to understand the larger context around an individual query in [performance tracing](./performance.html) or [error tracking](./error-tracking.html).

<h2 id="service">Full Service View</h2>

At the service level, you are able to view your schema's performance across your entire GraphQL service. This GraphQL specific view is useful for determining which types and fields are most used and their service times, which enables you to determine which resolvers require optimization. The typical service level schema view will appear as follows:

![Full Service Schema View](./img/schema-view/service-schema-view.png)

<h2 id="per-operation">Per Operation View</h2>

In addition to the full service level, Engine allows you to drill down further into GraphQL, providing schema performance per operation. This view provides context around which fields are causing a slowdown or erroring. After comparing the request rates, error information, or service times, you can then move to a specific trace view to determine the impact of an error or field over time, across the resolution of a query.

![Per Operation Schema View](./img/schema-view/operation-schema-view.png)

---
title: Response Caching
description: Speed up your GraphQL responses and reduce load on your backends by enabling caching in Engine.
---

API caching is a standard best practice, both to reduce the load on your servers, and to accelerate API responses and decrease page render times. But because GraphQL requests are often sent to a single endpoint with POST requests, existing HTTP caching solutions don’t work well for GraphQL APIs.

To bring caching to GraphQL, we've developed [Apollo Cache Control](https://github.com/apollographql/apollo-cache-control), a new open standard that allows servers to specify exactly what parts of a response can be cached, and for how long. Apollo Engine implements Apollo Cache Control, caching whole query responses based on a computed cacheability of each new query type, without requiring any manual configuration. Engine also includes a really nice way to see how cache policies impact each query type, making it easy to optimize your queries for cacheability, refine field-level cache policies, and pinpoint slow components on screen that could benefit from caching.

These are the the steps to configure response caching:

1. Enable cache control in your server and schema
1. Add the caching configuration to the Engine config file

> The initial preview of caching only supports Apollo Server, but we're working with the community to add support for Apollo Cache Control to other server libraries.

> If you are using `express-graphql`, we recommend you switch to Apollo Server to use caching. Both `express-graphql` and Apollo Server are based on the [`graphql-js`](https://github.com/graphql/graphql-js) reference implementation, and switching should only require changing a few lines of code.

<h2 id="enable-cache-control">Enabling cache control</h2>

Apollo Server includes built-in support for Apollo Cache Control from version 1.2.0 onwards.

The only code change required is to add `tracing: true` and `cacheControl: true` to the options passed to the Apollo Server middleware function for your framework of choice. For example, for Express:

```js
app.use('/graphql', bodyParser.json(), graphqlExpress({
  schema,
  context: {},
  tracing: true,
  cacheControl: true
}));
```

<h3 id="cache-hints">Adding cache hints</h3>

Cache hints can be added to your schema using directives on your types and fields. When executing your query, these hints will be added to the response and interpreted by Engine to compute a cache policy for the response. Hints on fields override hints specified on the target type.

```graphql
type Post @cacheControl(maxAge: 240) {
  id: Int!
  title: String
  author: Author
  votes: Int @cacheControl(maxAge: 30)
  readByCurrentUser: Boolean! @cacheControl(scope: PRIVATE)
}
```

If you need to add cache hints dynamically, you can use a programmatic API from within your resolvers.

```js
const resolvers = {
  Query: {
    post: (_, { id }, _, { cacheControl }) => {
      cacheControl.setCacheHint({ maxAge: 60 });
      return find(posts, { id });
    }
  }
}
```

If set up correctly, for this query:

```graphql
query {
  post(id: 1) {
    title
    votes
    readByCurrentUser
  }
}
```

You should receive cache control data in the `extensions` field of your response:

```js
"cacheControl": {
  "version": 1,
  "hints": [
    {
      "path": [
        "post"
      ],
      "maxAge": 240
    },
    {
      "path": [
        "post",
        "votes"
      ],
      "maxAge": 30
    },
    {
      "path": [
        "post",
        "readByCurrentUser"
      ],
      "scope": "PRIVATE"
    }
  ]
}
```

<h2 id="how-it-works">How caching works</h2>

Caching in Engine accepts cache control hints in a fine-grained way, but caches the entire result. Engine computes a cache privacy and expiration date by combining the data from all of the fields returned by the server for a particular request. It errs on the safe side, so shorter `maxAge` results override longer, and `PRIVATE` scope overrides public.

<h3 id="max-age-combination">maxAge aggregation</h3>

To determine the expiration time of a particular query, Engine looks at all of the `maxAge` hints returned by the server, and picks the shortest. For example, the above result indicates a `240` maxAge for one field, and `30` for another, which means that Engine will use `30` as the overall expiration time for the whole result.

The biggest benefit of collecting the cache hints on a per-field basis is that you can use the Engine UI to understand your cache hit rates and the overall `maxAge` for an operation. Here's what the caching view looks like:

TODO cache trace

<h3 id="public-vs-private">Public vs. private scope</h3>

TODO private data is cached per-user, watch out for memory

<h3 id="splitting-queries">Splitting queries to improve caching</h3>

TODO sometimes it makes sense to ask for public scoped data in a separate query so it can be cached, reducing load on your server

<h2 id="visualizing">Visualizing caching</h2>

One of the best parts about caching with Engine is that you can easily see how it's working once you set it up. It deeply integrates into the [performance tracing](./performance.html) views, so that you can understand how caching is helping you decrease your server response times. Here are what the charts in the performance view look like when you've successfully enabled caching:

![Cache rate on volume chart](./img/cache-rate.png)

The volume chart now shows how many of your requests hit the cache instead of the underlying server.

![Cache rate on heat map](./img/cache-histogram.png)

The histogram uses differently-colored bars to represent cache vs. non-cache requests. So you can easily see that the cached requests are much much faster, with Engine responding to those requests in microseconds rather than the 50-100 milliseconds it would take to hit the underlying server.

<h2 id="engine-cache-config">Configuring Engine</h2>

There are three fields in the Engine configuration that are particularly relevant when setting up response caching. Below is an example of a complete Engine config for caching, using the internal embedded cache. For production use, we suggest configuring Memcache so that you can share the cache across Engine instances.

```js
{
  "stores": [
    {
      "name": "embeddedCache",
      "inMemory": {
        "cacheSize": 10485760
      }
    }
  ],
  "sessionAuth": {
    "store": "embeddedCache",
    "header": "Authorization"
  },
  "queryCache": {
    "publicFullQueryStore": "embeddedCache",
    "privateFullQueryStore": "embeddedCache"
  },
}
```

Here's an explanation of these config fields:

<h3 id="config.stores">stores</h3>

This is an array of stores where Engine can put cached data. TODO

<h3 id="config.sessionAuth">sessionAuth</h3>

This is useful when you want to do per-session caching with Engine. To be able to cache results for a particular user, Engine needs to know how to identify a logged-in user. In this example, we've configured it to look for an `Authorization` header, so private data will be stored with a key that's specific to the value of that header.

<h3 id="config.queryCache">queryCache</h3>

This maps the types of result caching Engine performs to the stores you've defined in the `stores` field. In this case, we're using the same store for both public and private cached data.

<h2 id="memcache">Using Memcache</h2>

For production workloads, we suggest setting up Memcache instead of using the built-in embedded cache. This will give you much more control over memory usage and enable sharing the cache across multiple Engine proxy instances.

TODO more detail, how to set up Memcache and configure it?

---
title: Response Caching
description: Speed up your GraphQL responses and reduce load on your backends by enabling caching in Engine.
---

API caching is a standard best practice, both to reduce the load on your servers, and to accelerate API responses and decrease page render times. But because GraphQL requests are POSTed to a single endpoint, existing HTTP caching solutions don’t work well for GraphQL APIs.

To bring caching to GraphQL, we first developed [Apollo Cache Control](https://github.com/apollographql/apollo-cache-control), a new open standard that allows servers to specify exactly what parts of a response can be cached, and for how long. Apollo Engine implements Apollo Cache Control, caching whole query responses based on a computed cacheability of each new query type, without requiring any manual configuration. Engine also includes a really nice way to see how cache policies impact each query type, making it easy to optimize your queries for cacheability, refine field-level cache policies, and pinpoint slow components on screen that could benefit from caching.

These are the the steps to configure response caching:

1. Enable cache control in your server and schema
1. Add the caching configuration to the Engine config file

> The initial preview of caching only supports Apollo Server, but we're working with the community to add support for Apollo Cache Control to other server libraries.

> If you are using `express-graphql`, we recommend you switch to Apollo Server. Both `express-graphql` and Apollo Server are based on the [`graphql-js`](https://github.com/graphql/graphql-js) reference implementation, and switching should only require changing a few lines of code.

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

### Configure Engine for caching

```
{
  "apiKey": "...",
  "origins": [
    {
      "http": {
        "url": "http://localhost:4001/graphql"
      }
    }
  ],
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
  "frontends": [
    {
      "host": "127.0.0.1",
      "port": 4000,
      "endpoint": "/graphql"
    }
  ],
  "queryCache": {
    "publicFullQueryStore": "embeddedCache",
    "privateFullQueryStore": "embeddedCache"
  },
  "reporting": {
    "endpointUrl": "https://engine-report.apollographql.com",
    "debugReports": true
  },
  "logging": {
    "level": "DEBUG"
  }
}

```

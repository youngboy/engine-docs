---
title: CDN Integration
description: Getting content delivery networks to cache your GraphQL responses
---

Many high-traffic web services use content-delivery networks such as [Akamai](https://www.akamai.com/) or [Fastly](https://www.fastly.com/) to cache their content as close to their clients as possible. Engine makes it straightforward to use CDNs with GraphQL queries whose responses can be cached while still passing more dynamic queries through to your GraphQL server.

To use Engine behind a CDN, you need to be able to tell the CDN which GraphQL responses it's allowed to cache, and you need to make sure that your GraphQL requests arrive in a format that CDNs cache. Engine supports this via its [caching](./caching.html) and [automatic persisted queries](./auto-persisted-queries.html) features. This page explains the basic steps for setting up these features to work with CDNs; for more details on how to configure these features, see their respective pages.

<h2 id="enable-cache-control" title="1. Enable cache control">Step 1: Enable cache control in your Apollo Server</h2>

Just add `cacheControl: true` to your Apollo Server middleware invocation, such as `graphqlExpress`. See [the caching docs](./caching.html#enable-cache-control) for more details on enabling Cache Control. Ensure that you are running at least version 1.0.4 of `apollo-server-express` (or your Apollo Server module of choice). Your call should look something like:

```js
app.use(
  "/graphql",
  bodyParser.json(),
  graphqlExpress({
    schema,
    context: {},
    tracing: true,
    cacheControl: true
  })
);
```

<h2 id="cache-hints" title="2. Add cache hints">Step 2: Add cache hints to your GraphQL schema</h2>

Add cache hints to your GraphQL schema so that Engine knows which fields and types are cacheable and for how long. For example, to say that all fields that return an `Author` should be cached for 60 seconds, and that the `posts` field should itself be cached for 180 seconds, write this in your schema:

```graphql
type Author @cacheControl(maxAge: 60) {
  id: Int
  firstName: String
  lastName: String
  posts: [Post] @cacheControl(maxAge: 180)
}
```

See [the cache hints docs](./caching.html#cache-hints) for more details on cache hints, including how to specify hints dynamically inside resolvers, how to set a default `maxAge` for all fields, and how to specify that a field should be cached for specific users only (in which case CDNs should ignore it).

Once you've done these two steps, Engine will serve the HTTP `Cache-Control` header on fully cacheable responses, so that any CDN in front of Engine will know which responses can be cached and for how long! By "fully cacheable", we mean any response containing only data with non-zero `maxAge`; the header will refer to the minimum `maxAge` value across the whole response, and it will be `public` unless some of the data is tagged `scope: PRIVATE`. You should be able to observe this header in your browser's dev tools. Engine will also cache the responses in its own default public in-memory cache.

<h2 id="enable-apq" title="3. Enable persisted queries">Step 3: Enable automatic persisted queries</h2>

However, your GraphQL requests are still probably big POST requests, and most CDNs will only cache GET requests, and GET requests generally work best if the URL is of a bounded size. To work with this, enable Engine's automatic persisted queries support. This allows clients to send short hashes instead of full queries, and you can configure it to use GET requests for those queries.

To do this, update your **client** code. First, add the package to your app:

```
npm install apollo-link-persisted-queries
```

Then, add the persisted queries link to your Apollo Client before the HTTP link:

```js
import { createPersistedQueryLink } from "apollo-link-persisted-queries";
import { createHttpLink } from "apollo-link-http";
import { InMemoryCache } from "apollo-cache-inmemory";
import { ApolloLink } from "apollo-link";
import ApolloClient from "apollo-client";

ApolloLink.from([
  createPersistedQueryLink({ useGETForHashedQueries: true }),
  createHttpLink({ uri: "/graphql" })
]);

const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: link
});
```

Make sure not to leave out the `useGETForHashedQueries: true`. Note that your client will still use POSTs for mutations, because it's generally best to avoid GETs for non-idempotent requests.

If your GraphQL server will be hosted on a different origin from where it will be accessed, you'll need to tell Engine what [CORS headers](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) to send in order to use Automatic Persisted Queries (APQ). If this step applies to you, then you will have already needed to set up CORS headers inside your app. Generally Engine can just forward your GraphQL server's CORS response headers to clients, but some messages in the APQ protocol don't involve talking to your GraphQL server, so you will need to statically configure Engine instead. (We are hoping to make this more automatic in a future version.) You can do this with the `overrideGraphqlResponseHeaders` option in your `ApolloEngine` constructor:

```
const engine = new ApolloEngine({
  apiKey: process.env.ENGINE_API_KEY,
  frontends: [{
    overrideGraphqlResponseHeaders: {
      'Access-Control-Allow-Origin': '*',
    },
  }],
});
```

If all is well, you should be able to see in your browser's dev tools that queries are now being sent as GET requests, and are still receiving appropriate `Cache-Control` response headers.

<h2 id="setup-cdn" title="4. Set up your CDN">Step 4: Set up your CDN!</h2>

How exactly this works depends on exactly which CDN you chose. Configure your CDN to send requests to your Engine-powered GraphQL app. For some CDNs, you may need to specially configure your CDN to honor origin Cache-Control headers; for example, here is [Akamai's documentation on that setting](https://learn.akamai.com/en-us/webhelp/ion/oca/GUID-57C31126-F745-4FFB-AA92-6A5AAC36A8DA.html). If all is well, your cacheable queries should now be cached by your CDN! Note that requests served directly by your CDN will not show up in your Engine dashboard.

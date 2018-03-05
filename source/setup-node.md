---
title: Set up Engine with Node
sidebar_title: Node with Apollo Server
description: Get Engine running with your Node.js GraphQL server.
---

With just a few minutes of setup, you can supercharge your Node.js GraphQL server with Apollo Engine to get performance tracing, caching, error tracking, and more. Let's get started!

Our main supported server library is [Apollo Server](https://www.apollographql.com/docs/apollo-server/). Engine relies on GraphQL response extensions like [Apollo Tracing](./apollo-tracing.html) and [Apollo Cache Control](https://github.com/apollographql/apollo-cache-control) to work, which come integrated with Apollo Server. Apollo Server is easy to use with every popular Node.js server middlewares such as Express, Hapi, and Koa.

We encourage you to [contact us](mailto:support@apollodata.com) with feedback or help if you encounter problems running Engine in your app. You can also join us in the public [#engine Slack Channel](https://www.apollographql.com/#slack).

<h2 id="setup-guide">Setup guide for Express</h2>

To get Engine running with Apollo Server and Express, just follow these three quick steps. This guide assumes you're running your GraphQL server with the `express` web server package for Node. If you're using a different framework, all of the steps are the same except step 3, for which you should check out the [other servers](#not-express) section of this page.

<h3 id="apollo-server-config" title="Apollo Server Config">1. Enable Tracing and Cache Control in Apollo Server</h3>

Just add `tracing: true` and `cacheControl: true` to the options passed to the Apollo Server middleware function.

For example:

```javascript
const { graphqlExpress } = require('apollo-server-express');

app.use('/graphql', bodyParser.json(), graphqlExpress({
  schema,
  context: { ... },

  // Add the two options below
  tracing: true,
  cacheControl: true
}));
```

If you are using `express-graphql`, we recommend switching to Apollo Server. Both `express-graphql` and Apollo Server use the same [`graphql-js`](https://github.com/graphql/graphql-js) reference implementation, so your schema will work in exactly the same way. Switching should only require changing a few lines of code.

#### Check if it worked!

You can test that you've correctly configured Apollo Server's tracing and cacheControl extensions by running a query against your API using GraphiQL.

The `"tracing"` field should now be returned as part of the response extensions like below. Don't worry, this data won't make it to your frontend since Engine will filter it out.

```js
{
  "data": { ... },
  "extensions": {
    "tracing": { ... }
  }
}
```

If you see the `tracing` data in the result, move on to the next step!

<h3 id="api-key" title="Get API key">2. Get an Engine API key</h3>

[Log into Apollo Engine](http://engine.apollographql.com/) via your browser and create a service to get an API key. We'll pass it into our Apollo Engine constructor in the next step.

<h3 id="apollo-engine" title="Add Engine">3. Add Engine to your server</h3>

First, install the `apollo-engine` package from npm:

```bash
npm install --save apollo-engine
```

Then, import the `ApolloEngine` constructor at the top, create a new Engine instance, and replace `app.listen()` with `engine.listen()`. Engine will now wrap your endpoint and process GraphQL requests and responses. See the code sample below:

```js
// Import ApolloEngine
const { ApolloEngine } = require("apollo-engine");

// Initialize your Express app like before
const app = express();

// All of your GraphQL middleware go here
app.use('/graphql', ...);

// Initialize engine with your API key
const engine = new ApolloEngine({
  apiKey: 'API_KEY_HERE'
});

// Call engine.listen instead of app.listen(port)
engine.listen({
  port: 3000,
  expressApp: app
});
```

And you're done!

The code above is specifically for `express`. For other servers, check out the [other servers](#not-express) section of this page for the appropriate snippet.

#### Checking if it worked

Call your endpoint again with GraphiQL, and you should no longer see the `tracing` data in the result since Engine is now consuming it! Check the Engine UI for your new service, and you should see it confirm that the setup worked.

If you see the above, congratulations, you're up and running! Otherwise, look at some of the options below.

#### Configuring the GraphQL path

You can use the `graphqlPaths` option if your GraphQL API responds to requests on a path other than `/graphql`.

```js
// For engine.listen
engine.listen({
  port: 3000,
  expressApp: app,

  // GraphQL endpoint suffix - '/graphql' by default
  graphqlPaths: ['/graphql', '/special-graphql'],
});
```

#### Configuring debug logging

If you need some extra debugging information, you can add it with the `logging.level` option as below.

```javascript
// For the constructor
const engine = new ApolloEngine({
  apiKey: "API_KEY_HERE",
  logging: {
    level: "DEBUG" // Engine Proxy logging level. DEBUG, INFO, WARN or ERROR
  }
});
```

<h2 id="using-engine" title="Using Engine">Using Engine</h2>

Once your server is set up, navigate to your newly created Engine service in the [Engine UI](https://engine.apollographql.com). It should indicate that you've set everything up successfully. Start sending requests to your Node server to see performance metrics!

To get the most out of Engine, check out the documentation in the "Features" section on the left. In particular, a good next step might be to set up [response caching](./caching.html) which can make your GraphQL server much faster by storing commonly requested data, much like HTTP caching does.

<h2 id="not-express" title="Other Node servers">Node servers other than Express</h2>

If you're using one of the Node.js server packages that Apollo Server supports that aren't express, you've come to the right place! Find your server library below for an example.

All of these are just one line to set up, and the integration is built right into `ApolloEngine` like so:

#### Connect

Connect is the same as Express.

```js
const app = new connect();

engine.listen({
  port: 3000,
  expressApp: app
});
```

#### Koa

Just pass your server into the `koaApp` option:

```js
const app = new koa();

// Replace your app.listen with engine.listen
engine.listen({
  port: 3000,
  koaApp: app
});
```

#### Hapi

This is a little more complex but also built right in!

```js
TODO
```

#### Meteor

Just call `meteorListen` on the built-in `WebApp` server.

```js
engine.meteorListen(WebApp);
```

If you want to pass options to the Engine listener, you can pass them as the second parameter:

```js
engine.meteorListen(WebApp, { graphqlPaths: [ "/other-graphql" ]});
```

<h3 id="lambda">AWS Lambda</h3>

Since Engine relies on some state across requests to do performance tracing and caching, it needs to be run in a different way when you're using lambda that involves running a separate container. Thankfully, you can do this easily in just a few clicks!

Head on over to the [AWS Lambda setup guide](./setup-lambda.html) to learn how to do it.

<h2 id="api">API</h2>

If you need additional configuration, you've come to the right place.

<h3 id="api-apollo-engine" title="new ApolloEngine()">new ApolloEngine(config)</h3>

This is where you pass in configuration for how Engine should work. Even though you set up Engine as an npm package, it's actually a Go binary, which enables it to do things that would be harder to do in a performant way with purely Node.js code, and work the same way across many platforms.

You can find the complete set of configuration that Engine accepts in the [full API docs](./proto-doc.html) page, but here are some useful fields you should know about:

// TODO

<h3 id="api-engine.listen" title="engine.listen()">engine.listen(options, [callback])</h3>

The `listen` method is what actually starts your web server and asks its requests to go through Engine. It accepts a set of options and an optional callback that tells you when Engine and the server have started, or gives you an error:

```js
const app = express();
const engine = new ApolloEngine(config);

const port = 3000;

engine.listen({
  port,
  expressApp: app,
  // ... more options go here
}, () => {
  console.log(`Listening on port ${port}!`);
});
```

We intentionally made this funciton very similar to how `app.listen` would work, so that you can easily drop Engine into your existing app.

<h3 id="api-engine.on">Event: 'error'</h3>

Much like with the Express `app.listen`, you can find out about server or Engine start-up errors by listening for an `'error'` event on Engine, like so:

```js
const app = express();
const engine = new ApolloEngine(config);

engine.on('error', (err) => {
  console.log('There was an error starting the server or Engine.');
  console.error(err);

  // The app failed to start, we probably want to kill the server
  process.exit(1);
});

engine.listen(options);
```

If you don't handle this error event, much like with a regular Express app, it will kill your server with an unhandled error exception.

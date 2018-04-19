---
title: Set up Engine with Node
sidebar_title: Node with Apollo Server
description: Get Engine running with your Node.js GraphQL server.
---

With just a few minutes of setup, you can supercharge your Node.js GraphQL server with Apollo Engine to get performance tracing, caching, error tracking, and more. Let's get started!

Our main supported server library is [Apollo Server](https://www.apollographql.com/docs/apollo-server/). Engine relies on GraphQL response extensions like [Apollo Tracing](./apollo-tracing.html) and [Apollo Cache Control](https://github.com/apollographql/apollo-cache-control) to work, which come integrated with Apollo Server. Apollo Server is easy to use with every popular Node.js web framework such as Express, Hapi, and Koa.

We encourage you to [contact us](mailto:support@apollodata.com) with feedback or help if you encounter problems running Engine in your app. You can also join us in the public [#engine Slack Channel](https://www.apollographql.com/slack).

<h2 id="setup-guide">Setup guide for Express</h2>

To get Engine running with Apollo Server and Express, just follow these three quick steps. This guide assumes you're running your GraphQL server with the `express` web server package for Node. If you're using a different framework, all of the steps are the same except step 3, for which you should check out the [other servers](#not-express) section of this page.

<h3 id="apollo-server-config" title="Apollo Server config">1. Enable Tracing and Cache Control in Apollo Server</h3>

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

#### Check if Step 1 worked!

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
const { ApolloEngine } = require('apollo-engine');

// Initialize your Express app like before
const app = express();

// All of your GraphQL middleware goes here
app.use('/graphql', ...);

// Initialize engine with your API key. Alternatively,
// set the ENGINE_API_KEY environment variable when you
// run your program.
const engine = new ApolloEngine({
  apiKey: 'API_KEY_HERE'
});

// Call engine.listen instead of app.listen(port)
engine.listen({
  port: 3000,
  expressApp: app,
});
```

And you're done!

The code above is specifically for `express`. For other servers, check out the [other servers](#not-express) section of this page for the appropriate snippet.

#### Check if Step 3 worked

Call your endpoint again with GraphiQL, and you should no longer see the `tracing` data in the result since Engine is now consuming it! If it isn't working yet, try [the extra options documented below](#extra-setup-config).

<h2 id="using-engine" title="Using Engine">Using Engine</h2>

Once your server is set up, navigate to your newly created Engine service in the [Engine UI](https://engine.apollographql.com). It should indicate that you've set everything up successfully. Start sending requests to your Node server to see performance metrics!

To get the most out of Engine, check out the documentation in the "Features" section on the left. In particular, a good next step might be to set up [response caching](./caching.html) which can make your GraphQL server much faster by storing commonly requested data, much like HTTP caching does.

<h2 id="extra-setup-config">Extra options</h2>

If Engine is not yet working after the basic setup steps above, consider the two options below.

#### Configuring the GraphQL path

You can use the `graphqlPaths` option if your GraphQL API responds to requests on a path other than `/graphql`, or on more than one path.

```js
// For engine.listen
engine.listen({
  port: 3000,
  expressApp: app,

  // GraphQL endpoint suffix - '/graphql' by default
  graphqlPaths: ['/graphql', '/special-graphql'],
});
```

<h4 id="proxy-logging">Configuring the Proxy's internal logging</h4>

If Apollo Support suggests you need extra debugging information on Engine's internals, you can add turn it on by setting the `logging.level` option to `DEBUG` as below.  Alternatively, if you'd like to see less logs from Engine (such as only errors or warnings), set it it `ERROR` or `WARN`.

```javascript
// For the constructor
const engine = new ApolloEngine({
  logging: {
    level: "DEBUG" // Engine Proxy logging level. DEBUG, INFO (default), WARN or ERROR.
  },
});
```


<h2 id="deploy" title="Deploy">Deploy</h2>

Now that you have integrated Engine into your GraphQL server, you are ready to deploy your GraphQL service. Follow [these steps](./setup-heroku.html) to put your server into production on Heroku.

<h2 id="not-express" title="Other Node servers">Node servers other than Express</h2>

If you're using one of the Node.js server packages that Apollo Server supports that aren't express, you've come to the right place! Find your server library below for an example.

All of these are just one line to set up, and the integration is built right into `ApolloEngine` like so:

#### Connect

Follow the same instructions as for Express, but specify your app with the `connectApp` option instead.

```js
const app = new connect();

// Replace your app.listen with engine.listen
engine.listen({
  port: 3000,
  connectApp: app,
});
```

#### Koa

Follow the same instructions as for Express, but specify your app with the `koaApp` option instead.

```js
const app = new koa();

// Replace your app.listen with engine.listen
engine.listen({
  port: 3000,
  koaApp: app,
});
```

#### Restify

Follow the same instructions as for Express, but specify your app (which Restify calls a "server") with the `restifyServer` option instead.

```js
const server = restify.createServer(...);

// Replace your server.listen with engine.listen
engine.listen({
  port: 3000,
  restifyServer: server,
});
```

#### Micro

The app objects in the Micro framework are just instances of Node's built-in `http.Server`.

Follow the same instructions as for Express, but specify your server with the `httpServer` option instead.

```js
const server = micro(microRouter.post('/graphql', handler));

// Replace your server.listen with engine.listen
engine.listen({
  port: 3000,
  httpServer: server,
});
```


#### Hapi

This is a little more complex but also built right in! Hapi doesn't have a `listen` method, so you have to provide a special object to the `hapi.Server` constructor, and turn off auto-listening mode. Note that this assumes you are running the `engine.hapiListener` method inside an async function, as setting up Hapi servers generally requires calling `await` on a few async methods.

```js
const engine = new ApolloEngine({apiKey: 'API_KEY'});
const hapiListener = await engine.hapiListener({port: 3000});
const hapiServer = new hapi.Server({
  autoListen: false,
  listener: hapiListener,
});
// ... set up your server ...
await server.start();
```

The instructions above are for Hapi v17. The Hapi v16 API is slightly different (and also requires you to use a version of `apollo-server-hapi` older than 1.3.0). Engine is currently tested with Hapi v17, but we believe it also works with Hapi v16. We have a [separate example](https://github.com/jbaxleyiii/hapi-16-with-engine) showing how to use `apollo-server-hapi` and `ApolloEngine` with Hapi v16.

#### Meteor

Just call `meteorListen` on the built-in `WebApp` server at the top level of your app's server code:

```js
engine.meteorListen(WebApp);
```

If you want to pass options to the Engine listener, you can pass them as the second parameter:

```js
engine.meteorListen(WebApp, { graphqlPaths: [ "/other-graphql" ]});
```

#### Nest

Nest's listen function does two things, unlike Express/Connect/etc's which does one thing, so in order to decouple them you have to do this:

```js
// Replace your app.listen with engine.listen after awaiting app.init()
await app.init();
await engine.listen({
  port: PORT,
  httpServer: app.getHttpServer(),
});
```

#### Other Frameworks (and Node's built-in `http.Server`)

These instructions work for any framework not explicitly supported which gives you access to an `http.Server`, and for `http.Server` itself.

Follow the same instructions as for Express, but specify your server with the `httpServer` option instead.

```js
const http = require('http');

const server = http.createServer((request, response) => {
  // code goes here!
});

// Replace your server.listen with engine.listen
engine.listen({
  port: 3000,
  httpServer: server,
});
```

<h2 id="lambda">AWS Lambda</h2>

Since Engine relies on tracking some state across requests to do performance tracing and caching, it needs to be deployed in a different way when you're using Lambda that involves running a separate container. Thankfully, you can do this easily in just a few clicks!

Head on over to the [AWS Lambda setup guide](./setup-lambda.html) to learn how to do it.

<h2 id="https">Serving HTTPS and HTTP/2</h2>

If you'd like the Engine Proxy to serve HTTPS (aka "TLS") traffic directly from the Engine Proxy, you'll need to generate a public certificate and private key in PEM format, and point Engine at those files. You can optionally ask it to run an unencrypted HTTP server on another port that redirects all traffic to HTTPS.  This server will listen on the port that you pass to `engine.listen`.  When serving HTTPS, the Engine Proxy supports HTTP/2 as well as HTTP/1.1!

```js
const engine = new ApolloEngine({
  frontends: [{
    tls: {
      // This file should contain your site's certificate, followed by any intermediate
      // certificates (but not the root certificate) that go along with it.
      certificateFile: 'certificate.pem',
      // This file should contain your site's private key.
      keyFile: 'key.pem',
      // Optional: Engine will also listen on this port with an unencrypted HTTP
      // server and redirect all traffic to https.  Note that this must be specified
      // as a number, not a string.
      redirectFromUnencryptedPorts: [+(process.env.HTTP_REDIRECT_PORT)],
    }
  }],
});
engine.listen({port: process.env.HTTPS_PORT, expressApp: app});
```

If instead of a redirect you'd like the Engine Proxy to serve query results over both HTTP and HTTPS on different ports, you need to specify two separate frontend records:

```js
const engine = new ApolloEngine({
  frontends: [{
    // This frontend has no specific configuration and inherits its port from the
    // listen call.
  }, {
    // This frontend is the HTTPS server. Note that port inside the `new ApolloEngine()`
    // constructor needs to be a number, not a string..
    port: +(process.env.HTTPS_PORT),
    tls: {
      certificateFile: 'certificate.pem',
      keyFile: 'key.pem',
    }
  }],
});
engine.listen({port: process.env.HTTP_PORT, expressApp: app});
```


<h2 id="api">API</h2>

If you need additional configuration, you've come to the right place.

<h3 id="api-apollo-engine" title="ApolloEngine">new ApolloEngine(config)</h3>

The `ApolloEngine` class is the main API for running the Engine Proxy and integrating it with your Node web framework. This is where you pass in configuration for how Engine should work. Even though you set up Engine as an npm package, it's actually a Go binary, which enables it to do things that would be harder to do in a performant way with purely Node.js code, while working the same way across many platforms.

To get started, the only configuration field you need to specify is `apiKey`: the Engine API key copied from the Engine website. (You can leave this out if you specify the API key in the `ENGINE_API_KEY` environment variable instead.)

You can find the complete set of configuration that Engine accepts in the [full API docs](./proxy-config.html) page, but here are some commonly-used fields worth knowing about:

```js
const engine = new ApolloEngine({
  // The only mandatory field!  (Can be set via ENGINE_API_KEY instead.)
  apiKey: 'my-api-key',

  // Specify behavior for how the Engine Proxy should connect to the
  // GraphQL origin (your Node GraphQL server). While the Proxy does
  // support multiple origins, most users will only put one origin
  // in this array.
  origins: [{
    // If you are using the apollo-link-batch-http package to combine
    // multiple GraphQL requests into a single HTTP request, set this
    // to ensure that the Engine Proxy sends them to your Node web
    // server as a single request as well. (If you don't set this,
    // the Proxy will split it apart into individual HTTP requests.)
    supportsBatch: true,
    // Amount of time to wait for your Node GraphQL server to return
    // a response before timing out. Defaults to "30s". Specified as
    // a string containing a number and a unit such as "ms" or "s".
    requestTimeout: "60s",
    // HTTP-specific options. (The options above also apply to other
    // origin types such as Lambda.)
    http: {
      // The Engine Proxy will set the following request headers on
      // all requests it makes to your GraphQL server.
      overrideRequestHeaders: {
        // By default, the Engine Proxy preserves the Host header
        // from the user's request.
        host: 'mysite.example.com',
      },
    }
  }],

  // Specify behavior for how the Engine Proxy listens as an HTTP
  // server. While the Proxy does support multiple listening
  // frontends, most users will only put one frontend in this array.
  frontends: [{
    // Specify behavior for handling GraphQL extensions in GraphQL
    // responses.
    extensions: {
      // Extensions which will never be returned to clients.
      // Defaults to ['tracing'].
      blacklist: ['tracing', 'cacheControl'],
      // Extensions to only return to clients if the client requests
      // them.  Defaults to `['tracing', 'cacheControl']`.
      strip: ['tracing'],
    },
  }],

  // Resize the default in-memory cache.
  stores: [{
    inMemory: {
      cacheSize: 104857600,  // 100 MB; defaults to 50MB.
    },
  }],
  sessionAuth: {
    // Use the value of the 'session-id' cookie to isolate responses
    // in the private full query cache from those from other sessions.
    cookie: 'session-id',
  },
  queryCache: {
    // Turn off the public full query cache. The cache is only used for
    // responses with the 'cache-control' GraphQL extension.
    publicFullQueryStore: 'disabled',
  },

  logging: {
    // Only show warnings and errors, not info messages.
    level: 'WARN',
  },
});
```

You can also specify config as a filename pointing to a JSON file.

<h3 id="api-engine.listen" title="engine.listen()">engine.listen(options, [callback])</h3>

The `listen` method is what actually starts your web server and asks its requests to go through Engine. You use it for all supported web frameworks except for Hapi and Meteor.

Just like the `listen` method on `http.Server` and most web framework app objects, it accepts a set of options and an optional callback that tells you when Engine and the server have started, or emits an error:

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

We intentionally made this function very similar to how `app.listen` would work, so that you can easily drop Engine into your existing app.  Like `app.listen`, the callback is only called on success.

All calls to `listen` must provide the `port` option and exactly one of the options that specify your app (`expressApp`, `connectApp`, `koaApp`, `restifyServer`, or `httpServer`).

The `port` option must be a number (or a string containing a number) specifying the TCP port on which the Engine Proxy will listen.  (Engine does not currently support listening on Unix domain sockets.)  `listen` then starts your app's own web server on an ephemeral port.  All HTTP traffic for you app will go directly to the Engine Proxy binary, which will proxy it to your app.  Traffic on paths other than those specified in the `graphqlPaths` option to `listen` will be sent through without modification. The Engine Proxy supports transparently proxying Websockets as well as normal HTTP traffic.

If you set `port` to 0, the Engine Proxy will listen on an ephemeral port. Once the listen callback is called, you can retrieve the port it listened on with `engine.engineListeningAddress.port`. This is useful for running Engine dynamically from tests.

In addition to `port` and the options that specify your app, `listen` has some optional options:

- `graphqlPaths` (non-empty array of strings): Specifies the URL paths which the Engine Proxy will treat as GraphQL instead of proxying transparently without parsing. Defaults to `['/graphql']`.
- `host` (string): The interface on which the Engine Proxy will accept connections; same as the `host` argument to `net.Server.listen`. Defaults to listening on all interfaces. For example, if your machine is publically accessible on the Internet but you only want processes on your machine to be able to connect to the Engine Proxy, specify the "loopback address" `'127.0.0.1'`.
- `innerHost` (string): The interface on which your Node server will accept connections from the Engine Proxy. Defaults to `'127.0.0.1'` (the inner Node server is only accessible from your machine).
- `launcherOptions` (object): Options which control how we launch the Engine Proxy binary. Note that these are the same as the arguments to [the `start` method of the `ApolloEngineLauncher` class](./setup-standalone.html#api-launcher.start).
- `launcherOptions.startupTimeout` (number): How many milliseconds to wait for the Engine Proxy to successfully start up and listen before timing out. Defaults to 5000 (5 seconds). Set to 0 to wait forever.
- `launcherOptions.proxyStderrStream` ([writable stream](https://nodejs.org/api/stream.html#stream_writable_streams)): By default, the Engine Proxy binary's standard error stream is written to your Node process's standard error. Set this to a writable stream to process it in a different way.  (By default, the Engine Proxy writes its logs to standard error at log level `INFO`. If all you want to do is hide the `INFO` logs, try [configuring the log level](#proxy-logging) instead.)
- `launcherOptions.proxyStdoutStream` ([writable stream](https://nodejs.org/api/stream.html#stream_writable_streams)): By default, the Engine Proxy binary's standard output stream is written to your Node process's standard output. Set this to a writable stream to process it in a different way.  (By default, the Engine Proxy does not write anything to standard output.)
- `launcherOptions.processCleanupEvents` (array of strings): By default, `engine.listen()` registers handlers for the `exit`, `uncaughtException`, `SIGINT`, `SIGTERM`, and `SIGUSR2` [events on `process`](https://nodejs.org/api/process.html#process_process_events) to kill the Engine Proxy process (and, for the signal handlers, to wait for it to exit before resignaling the Node process). You can customize the list of events by specifying the event names as this option, if you want Engine to handle more signals or to let you do something different on the default events.


<h3 id="api-engine.on">Event: 'error'</h3>

Much like with the Express `app.listen`, you can find out about server or Engine start-up errors by listening for an `'error'` event on Engine, like so:

```js
const app = express();
const engine = new ApolloEngine(config);

engine.on('error', err => {
  console.log('There was an error starting the server or Engine.');
  console.error(err);

  // The app failed to start, we probably want to kill the server
  process.exit(1);
});

engine.listen(options);
```

If you don't handle this error event, much like with a regular Express app, it will kill your server with an unhandled error exception.

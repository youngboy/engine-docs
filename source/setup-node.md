---
title: Set up Engine with Node
sidebar_title: Node with Apollo Server
description: Get Engine running with your Node.js GraphQL server.
---

This guide will help you set up Engine with Node.js. Because Engine relies on some GraphQL response extensions like Tracing and Cache Control, the main supported server library is [Apollo Server](https://www.apollographql.com/docs/apollo-server/), which you can use with every popular Node.js server middleware such as Express, Hapi, Koa, Restify, and more.

The guide below will get you up and running as fast as possible. For advanced functionality and other features check out the other articles in the docs.

<h2 id="enable-apollo-tracing-and-cache-control" title="1. Tracing + Cache Control">1. Enable Apollo Tracing and Cache Control</h2>

With Apollo Server, this part of the setup is trivial: The only code change required is to add `tracing: true` and `cacheControl: true` to the options passed to the Apollo Server middleware function for your framework of choice. For example, with Express:

```javascript
app.use('/graphql', bodyParser.json(), graphqlExpress({
  schema,
  context: { ... },

  // Add the two options below
  tracing: true,
  cacheControl: true
}));
```

If you are using `express-graphql`, we recommend switching to Apollo Server, which includes more features out of the box. Both `express-graphql` and Apollo Server use the same [`graphql-js`](https://github.com/graphql/graphql-js) reference implementation, so your schema will work in exactly the same way. Switching should only require changing a few lines of code.

<h3 id="enabling-compression">1.1 Enable compression</h3>

Once instrumented, the tracing package will increase the size of responses traveling between your GraphQL API and the Engine proxy, because the requests will be augmented with additional tracing data.

Because of this, we recommend that you enable gzip compression in your GraphQL server, since the added volume from the tracing format compresses well.

**Express**

Install the compression middleware (https://github.com/expressjs/compression) package into your app with `npm install compression --save` and add it to your server's middleware stack as follows:

```javascript
var compression = require('compression')
app.use(compression());
```

**Koa**

Install the compress middleware (https://github.com/koajs/compress) package into your app with `npm install koa-compress --save` and add it to your server's middleware stack as follows:

```javascript
var compress = require('koa-compress')
app.use(compress())
```

**Hapi**

Hapi comes with support for compression enabled by default, unless it has been configured with `compression: false`.

<h2 id="configure-proxy">2. Add Engine middleware</h2>

We provide an NPM package that includes a pre-built copy of the Engine proxy, which makes initial setup with Node very simple. It spawns an Engine process side-by-side with your GraphQL server process and incoming GraphQL operations are routed through the Engine proxy and then sent to your server.

<h3 id="initialize">2.1. Initialize Engine</h3>

First, install the Engine package from npm:

```bash
npm install --save apollo-engine
```

Then, import the `Engine` constructor at the top, create a new Engine instance, and call `.start()` on it:

```js
import { Engine } from 'apollo-engine';

const engine = new Engine({
  engineConfig: {
    apiKey: 'API_KEY_HERE'
  }
});

engine.start();
```

It does not matter when you call `engine.start()` in your server file, but the earlier Engine is started the better. Your server will start normally and handle requests without the Engine proxy until it has fully started and is ready.

#### Optional configuration

If you need some extra debugging information or customizability, you can add it with the options below:

```javascript
const engine = new Engine({
  engineConfig: {
    apiKey: engineApiKey,
    logging: {
      level: 'DEBUG'   // Engine Proxy logging level. DEBUG, INFO, WARN or ERROR
    }
  },

  // GraphQL port
  graphqlPort: process.env.PORT || 8003,

  // GraphQL endpoint suffix - '/graphql' by default
  endpoint: '/graphql',

  // Debug configuration that logs traffic between Proxy and GraphQL server
  dumpTraffic: true
});
```

<h3 id="api-key" title="2.2. Get API key">2.2. Get your Engine API key</h3>

You can get your `ENGINE_API_KEY` by creating a service in the [Engine UI](http://engine.apollographql.com/). You will need to log in and click "Add Service" to recieve your API key.

<h3 id="add-middleware" title="2.3. Add middleware">2.3. Add the middleware to your app</h3>

Add the Engine middleware to your app's middleware stack so that your app can route requests through the Engine proxy. Since the Engine middleware needs to process the raw requests to your server before they receive any other modifications, it's important that this is your app's _first middleware_.

The `apollo-engine` package supports the following middlewares:

1. `expressMiddleware` – use for Express servers.
2. `connectMiddleware` – use for Connect or Restify servers.
3. `instrumentHapiServer` – use for Hapi servers.
4. `koaMiddleware` – use for Koa servers.

In an Express server, adding the Engine middleware would look like this:

```javascript
const app = express();

app.use(engine.expressMiddleware());

// Other middleware
```

<h2 id="view-metrics-in-engine" title="3. View data">3. View your data in Engine</h2>

Once your server is set up, navigate to your newly created Engine service in the [Engine UI](https://engine.apollographql.com). It should indicate that you've set everything up successfully. Start sending requests to your Node server to start seeing performance metrics!

<h2 id="single-proxy-with-sidecar" title="Single proxy">Advanced: Single proxying sidecar configuration</h2>

An alternative to using the middleware to selectively forward requests to Engine and then back to your application is to proxy all traffic via the Engine proxy. This is more performant for pure or close to pure GraphQL workloads but will result in the proxy being in the request path even for non-GraphQL requests.

It also requires the ability to change the listening port of your application so Engine can instead listen on that port. You will need to then configure Engine with the new port of your application.
Whilst this requires a bit more knowledge of TCP ports and how to configure your application server stack it's the preferred configuration for higher performance GraphQL servers.

To use this mode of operation first remove the Engine middleware for your server if you added it above.
Then configure Engine like so:

```js
engine = new Engine({
  endpoint: '/graphql',
  graphqlPort: APP_PORT, // The port your application is listening on
    frontend: {
      host: '0.0.0.0', // Listen on all addresses
      port: ENGINE_PORT, // The port that Engine will listen on
      endpoint: '/graphql'
    }
});
```

Where `APP_PORT` is the port your app is now listening on and `ENGINE_PORT` is the port on which Engine proxy will listen. i.e If your application was originally listening on `3000` you would set `ENGINE_PORT` to be `3000` and `APP_PORT` to be a different port, say `3001` as an example and modify your app server to listen on this port.

In the case of `express` you can configure your apps listen port (`APP_PORT`) like so:

```js
var express = require("express");
var app = express();
// setup routes, middleware etc here
app.listen(APP_PORT)
```

<h2 id="standalone-docker-container" title="Docker container setup">Advanced: Standalone Docker container setup</h2>

This option involves running a standalone docker container that contains the Engine proxy process and is hosted and managed separately from your Node server. This is the best option to select when you need more control over the scaling, operation, and deployment of Engine, and also delivers better performance than running the proxy as a child process of your Node server.

<h3 id="create-config-json" title="1. Create config.json">1. Create a config.json file</h3>

The proxy container uses a JSON object to get configuration information. If the configuration is passed the path to your file, that file will be watched for changes. Changes will cause the proxy to adopt the new configuration without downtime.

Create a JSON configuration file that looks like this:

```js
{
  "apiKey": "<ENGINE_API_KEY>",
  "logging": {
    "level": "INFO"
  },
  "origins": [
    {
      "http": {
        "url": "http://localhost:3000/graphql"
      }
    }
  ],
  "frontends": [
    {
      "host": "0.0.0.0",
      "port": 3001,
      "endpoint": "/graphql"
    }
  ]
}
```

You can get your `ENGINE_API_KEY` by creating a service in the [Engine UI](http://engine.apollographql.com/). You will need to log in and click "Add Service" to recieve your API key.

**Configuration options:**

1. `apiKey`: The API key for the Engine service you want to report data to.
2. `logging.level` : Logging level for the proxy. Supported values are `DEBUG`, `INFO`, `WARN`, `ERROR`.
3. `origin.http.url` : The URL for your GraphQL server.
4. `frontend.host` : The hostname the proxy should be available on.
5. `frontend.port` : The port the proxy should bind to.
6. `frontend.endpoint` : The path for the proxy's GraphQL server . This is usually `/graphql`.

<h3 id="run-proxy" title="2. Run the Docker proxy">2. Run the Docker container proxy</h3>

The Engine proxy is a Docker image that you will deploy and manage separate from your server.

If you have a working [Docker installation](https://docs.docker.com/engine/installation/), use the following shell commands (variables replaced with the correct values for your environment) to run the Engine proxy:

```bash
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine:2018.01-43-g1747440e6
```

You can deploy and manage your Engine proxy anywhere Docker containers can be hosted. We run our own on Amazon's [EC2 Container Service](https://aws.amazon.com/ecs/).

We recognize that almost every team using Engine has a slightly different deployment environment, and encourage you to [contact us](mailto: support@apollodata.com) with feedback or for help if you encounter problems running the Engine proxy.

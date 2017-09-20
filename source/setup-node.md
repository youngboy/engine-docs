---
title: Setup for Node Servers
order: 1
---

**Supported Node servers:** [Apollo Server](https://github.com/apollographql/apollo-server) (Express, Hapi, Koa, Restify, and Lambda); [Express-GraphQL](https://github.com/graphql/express-graphql)

To get started with Engine, take the following steps:
1. Install the language-specific side-loader package **or** run the standalone docker container
2. Instrument your GraphQL server with Apollo tracing
3. Deploy your server – you're all set up!

## 1. Install the Proxy
There are two options for configuring and deploying the Engine proxy with Node servers. You can either install Engine's JavaScript side-loader package from NPM or run a standalone docker container.

### Side-Loader Package

We provide an NPM package that includes a pre-built copy of the Engine proxy. It spawns an Engine process side-by-side with your GraphQL server process. Incoming GraphQL operations are routed through the Engine proxy and then sent to your server.

#### Install the npm package

```
npm install --save https://s3.us-east-2.amazonaws.com/apollo-engine-deploy/apollo-engine-0.3.5.tgz
```

#### Adding Engine to the Node.js Server

The Engine proxy uses a JSON object to get configuration information. You can instrument your Node server code to support Engine by adding the following steps to the **top** of your server's code:

##### Step 1: Import Engine

```
import { Engine } from 'apollo-engine';
```

##### Step 2: Create a new Engine instance

When you instantiate Engine, you have two options for referencing configuration fields.

1. You can set the engine configuration directly with a json object. See “JSON Object”.
2. Or, create a new engine instance that uses a config.json file. See “Config.json”.

###### JSON OBJECT
```
const engine = new Engine({ engineConfig: { "apiKey": "<ENGINE_API_KEY>" } });
```
###### CONFIG.JSON
```
const engine = new Engine({ engineConfig: 'path/to/config.json' });
```

##### Step 3 (Optional): Configure Endpoint and Port

If you want to change the endoint or port that the Engine proxy is available at, you can set these optional configurations:

```
{
    engineConfig: string | EngineConfig, // Engine Configuration filename or object
    endpoint?: string,                   // GraphQL endpoint suffix - '/graphql' by default
    graphqlPort?: number                 // GraphQL port - 'process.env.PORT' by default
}
```

##### Step 4: Add engine.start();
```
engine.start();
```
##### Step 5: Invoke your Node.js middleware function

**Important**: This must be first middleware call. Besides this requirement, it does not matter when you call engine.start in your server file, **as long as it is before the other middleware**. The earlier Engine is started the better. Since `apollo-engine` acts as a proxy, it must be added to the middleware that actually processes the query. Your server will start normally and handle requests without the engine proxy until engine is ready.

Choose from expressMiddleware(), connectMiddleware(), instrumentHapiServer() or koaMiddleware()

For example, when using Express:
```
app.use(engine.expressMiddleware());

// ...
// other middleware / handlers
// ...
```

### Standalone Docker Container

Skip to [the instructions](/standalone-proxy.html) for installing the standalone docker container, which are the same regardless of which language your server is written in.

## 2. Instrument Apollo Tracing

Configure tracing in your GraphQL server to enable Engine to receive performance traces for your GraphQL requests. The instructions for Apollo Server and Experss-GraphQL specifically are below.

### Using Apollo Server

[Apollo Server](https://github.com/apollographql/apollo-server) has included built-in support for tracing since version `1.1.0`.

The only code change required to enable tracing is to add `tracing: true` to the options passed to Apollo Server's middleware function. For example, if you're using Express:
```
app.use('/graphql', bodyParser.json(), graphqlExpress({
  schema,
  context: {},
  tracing: true,
}));
```
This exact syntax may vary slightly depending on which server framework you're using (Express, Hapi, Koa, etc).

### Using Express-GraphQL

Enabling tracing with [Express-GraphQL](https://github.com/graphql/express-graphql) requires these additions to your server code:

```
import {
  TraceCollector,
  instrumentSchemaForTracing,
  formatTraceData
} from 'apollo-tracing';

...

app.use('/graphql', 
  (req, res, next) => {
    const traceCollector = new TraceCollector();
    traceCollector.requestDidStart();
    req._traceCollector = traceCollector;
    next(); 
  }, 
  graphqlHTTP(request => ({
    schema: instrumentSchemaForTracing(schema),
    context: {
      _traceCollector: request._traceCollector
    },
    graphiql: true,
    extensions: () => {
      const traceCollector = request._traceCollector;
      traceCollector.requestDidEnd();
      return {
        tracing: formatTraceData(traceCollector)
      }
    }
  }))
);
```

For more information, see the Github project: https://github.com/apollographql/apollo-tracing-js.

### Enabling Compression [Optional]

We recommend that you enable gzip compression in your GraphQL server, because the tracing format adds volume to the request response size but compresses well.

**Express**

Install the compression middleware (https://github.com/expressjs/compression) package into your app with "npm install compression --save" and add it to your server's middleware stack as follows:

```
var compression = require('compression')

app.use(compression());
```

**Koa**

Install the compress middleware (https://github.com/koajs/compress) package into your app with "npm install koa-compress --save" and add it to your server's middleware stack as follows:

```
var compress = require('koa-compress')

app.use(compress())
```

**Hapi**

Hapi comes with support for compression enabled by default, unless it has been configured with compression: false.

## 3. View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!

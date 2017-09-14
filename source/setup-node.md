---
title: Configure Engine for Node Servers
order: 1
---

We provide an NPM package that includes a pre-built copy of the Engine proxy. It spawns an Engine process side-by-side with your GraphQL server process. Incoming GraphQL operations get routed through the Engine proxy and then into your server.

The steps you'll follow to configure Engine are:

1. Install the NPM Engine package.
2. Install Apollo tracing for your GraphQL server
3. Deploy your server - you are then all set up!

## Install the NPM Engine agent

Install the npm package:

```
npm install --save https://s3.us-east-2.amazonaws.com/apollo-engine-deploy/apollo-engine-0.3.0.tgz
```

The Engine proxy uses a JSON object to get configuration information. 
Instrument your Node server code to support Engine by adding the following to the top of your server script:

```
import { Engine } from 'apollo-engine';

// create new engine instance from JS config object
const engine = new Engine({ engineConfig: { "apiKey": "<ENGINE_API_KEY>" } });

// OR create new engine instance from file
const engine = new Engine({ engineConfig: 'path/to/config.json' });

await engine.start();
// Invoke the function that corresponds to your Node Middleware. 
// Choose from expressMiddleware(), connectMiddleware(), instrumentHapiServer() or koaMiddleware()
// For example, when using Express:
app.use(engine.expressMiddleware());

// ...
// other middleware / handlers
// ...

```

If you want to change the endoint or port that the Engine proxy is available at, you can set these optional configurations:

```
{
    engineConfig: string | EngineConfig, // Engine Configuration filename or object
    endpoint?: string,                   // GraphQL endpoint - '/graphql' by default
    graphqlPort?: number                 // GraphQL port - 'process.env.PORT' by default
}
```

## Install Apollo Tracing

Install Apollo tracing in your GraphQL server to enable Engine to receive performance traces for your GraphQL requests.

### Using Apollo Server

Apollo Server includes built-in support for tracing from version 1.1.0 onwards.

The only code change required is to add "tracing: true" to the options passed to the Apollo Server middleware function for your framework of choice. For example, for Express:

```
app.use('/graphql', bodyParser.json(), graphqlExpress({
  schema,
  context: {},
  tracing: true,
}));
```

### Using express-graphql

Using Apollo Tracing with Express GraphQL requires these additions to your server code:

```
import {
  TraceCollector,
  instrumentSchemaForTracing,
  formatTraceData
} from 'apollo-tracing'

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

For more information, see the Github project: https://github.com/apollographql/apollo-tracing-js

### Enable gzip compression [Optional]

We recommend that you enable gzip compression in your GraphQL server, because the tracing format adds to the response size but compresses well.

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

## View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!

---
title: Upgrade from Optics Agent
order: 11
---

Apollo Engine is the next generation of Optics with new features like error reporting, query caching, and more. To enable support for more GraphQL server languages and unlock features like caching, Engine is built upon a proxy architecture. To upgrade from Optics to Engine you'll need to add the Engine proxy to your stack.

If you're already using Optics, there's an easy upgrade from your current Optics integration to Engine --- it's just a few lines of code in your `server.js` and an NPM package upgrade! If you are interested in using other languages, please see our other documentation pages.

**This** guide assumes you are starting with **a Node.js app **that has Optics already instrumented.

<h2 id="remove-optics-agent-instrumentation" title="Remove Optics Agent integration">Remove Optics agent integration</h2>

You can remove the code added to your Node `server.js` file to instrument Optics.

You no longer need to wrap your GraphQL schema in the Optics Agent instrumentation, so remove this line and remove your import of the `OpticsAgent`.

```javascript
// Remove your import of the OpticsAgent
// import OpticsAgent from 'optics-agent';

// Remove OPTICS_API_KEY reference

graphqlExpress((req) => {
  // Change this line back to this:
  // schema: OpticsAgent.instrumentSchema(schema)
  schema: schema

  // other options...
})
```

Remove the Optics Agent HTTP middleware:

```javascript
// Remove this line:
// expressServer.use(OpticsAgent.middleware());
```

Remove the `opticsContext` field from your GraphQL context.

```javascript
graphqlExpress((req) => {
  context: {
    // Remove the opticsContext field from your GraphQL context:
    // opticsContext: OpticsAgent.context(req),

    // other context fields...
  }
})
```

Remove the Optics Agent NPM package:

```
npm remove optics-agent
```

<h2 id="install-and-configure-engine" title="Install and Configure Engine">Install and Configure Engine</h2>

We recommend that you use Apollo Server. It's a much simpler integration.

<h3 id="get-api-key" title="Get your API Key">Get an API key from Engine</h3>

Log in to [engine.apollographql.com](https://engine.apollographql.com) and click "Add Service" in the upper right-hand corner.

Name your endpoint and save your **API key**.

<h3 id="enable-apollo-tracing" title="Enable Apollo Tracing">Enable Apollo Tracing</h3>

**If using Apollo Server**

The only code change required is to add `tracing: true` and `cacheControl: true` to the options passed to the Apollo Server middleware function for your framework of choice. For example, for Express:

```javascript
app.use('/graphql', bodyParser.json(), graphqlExpress({
  schema,
  context: {},
  // Enable tracing:
  tracing: true,
  cacheControl: true,
}));
```

**If using Express-GraphQL**

Using Apollo Tracing with express-graphql requires more manual configuration. See [this section](https://github.com/apollographql/apollo-tracing-js#express-graphql) of the docs for details.

<h3 id="add-engine-middleware" title="Add Engine Middleware">Add Engine framework integration code to your server</h3>

Import the `ApolloEngine` constructor from the apollo-engine NPM package.

```
npm install --save apollo-engine
```

```javascript
import { ApolloEngine } from 'apollo-engine';
```

Create a new Engine instance. Set the engine configuration, likely just your API key.  If you had lots of extra configuration in Optics, look at the [main Node setup instructionspage](./setup-node.html) to see where the equivalent options go now.

```javascript
const engine = new ApolloEngine({ apiKey: '<ENGINE_API_KEY>' });
```

Find the line where your Express server `listen`s. (This wasn't a line that Optics instrumented directly.)  (If you're not using Express, follow the [instructions on the main Node setup page](./setup-node.html#not-express).)

```javascript
// Replace this line:
app.listen(3000);

// With this line:
engine.listen({port: 3000, expressApp: app});
```

Note that if your app serves GraphQL on a path other than `/graphql`, you'll need to specify the `graphqlPaths` option.

<h2 id="test" title="Test">Run in localhost to test</h2>

Start the server in your localhost development environment.

Run your GraphQL queries and check that they provide the results you expect.

Check that they show up in your service report on engine.apollographql.com (http://engine.apollographql.com/).

<h2 id="deploy" title="Deploy"> Deploy to Staging and Production</h2>

Deploy the Node server to staging and then production!

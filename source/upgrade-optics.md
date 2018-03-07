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

<h2 id="install-and-configure-engine">Install and configure engine</h2>

[Read the Apollo Engine setup guide for Node.js to get Engine running!](setup-node.html)

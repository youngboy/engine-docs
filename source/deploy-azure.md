---
title: Deploying Node to Azure
description: How to configure and deploy your GraphQL server with Engine to Azure
---

Before starting, make sure you have [added Engine to your Node server](setup-node.html). 
> This guide is specifically made for Node GraphQL servers. For non-Node servers, we recommend using the 
[launcher API](https://deploy-preview-170--engine-docs.netlify.com/docs/engine/setup-standalone.html#apollo-engine-launcher)
which is a small Node script packaged with [`apollo-engine`](https://www.npmjs.com/package/apollo-engine) used to run Engine as a standalone proxy
server.

Because of how Azure maps incoming TCP ports to 
[Windows named pipes](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365590.aspx)
internally, you will need to configure Engine to listen on a named pipe rather than
on a port number. In order to do this, you can specify `pipePath` in the call to
`engine.listen({pipePath: process.env.PORT})`. Of course, this route alone will 
not work if you want to listen on TCP ports in development. For that use case,
follow the below snippet, provided by a community member:

```js
const server = createServer(app); 
server.setTimeout(30 * 1000); // 30 seconds

if (APOLLO_ENGINE_APIKEY) {
  const engine = new ApolloEngine({ apiKey: APOLLO_ENGINE_APIKEY });

  // Locally listen on a TCP port
  if (ENVIRONMENT !== 'production') { 
    engine.listen({ port: PORT as number || 3000, httpServer: server}, () => { 
      onListenCallback(server, modulesContext); 
    });
  // On azure the PORT variable will be a Windows named pipe, like `\\.\pipe\foo`
  } else { 
    engine.listen({ pipePath: PORT as any, httpServer: server }, () => { 
      onListenCallback(server, modulesContext); 
    });
  }
} else {
  server.listen({ port: PORT as any }, () => { 
    onListenCallback(server, modulesContext);
  });
}
```
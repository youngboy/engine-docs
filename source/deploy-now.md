---
title: Deploying Node to Zeit Now
description: How to configure and deploy your GraphQL server with Engine to Zeit Now
---

Before starting, make sure you have [added Engine to your Node server](setup-node.html). 
> This guide is specifically made for Node GraphQL servers. For non-Node servers, we recommend using the 
[launcher API](https://deploy-preview-170--engine-docs.netlify.com/docs/engine/setup-standalone.html#apollo-engine-launcher)
which is a small Node script packaged with [`apollo-engine`](https://www.npmjs.com/package/apollo-engine) used to run Engine as a standalone proxy
server.

[Zeit's now](https://zeit.co/now) has three different deployment options: a static website, Node.js (the default when you have a `package.json` file), and a Docker app. Engine uses two ports, which their Node.js deployment [doesn't support](https://zeit.co/docs/deployment-types/node#port-selection). For this reason, we recommend using the [Docker deployment](https://zeit.co/docs/deployment-types/docker).

First, we add `--docker` to our `now` deployment command, which we'll keep in our `package.json`. Note that we'll also set the PORT and ENGINE_API_KEY environment variables for Engine to rely on. 

`package.json`

```json
{
  ...
  "scripts": {
    "start": "node server.js",
    "deploy": "now --docker -e PORT=443 -e ENGINE_API_KEY=@engine-api-key -e NODE_ENV=production"
  }
}
```

We set the `ENGINE_API_KEY` using `now secrets`, which we set up with:

```
now secrets add engine-api-key "[API key]"
```

(You can find your API key under engine.apollographql.com/service/[yourservicename]/settings.)

We use `process.env.PORT` when calling `engine.listen()`. For instance:

```js
engine.listen({
  port: process.env.PORT,
  expressApp: app,
});
```

The port number from our deploy command should match our `Dockerfile` (which we place in the root directory):

```dockerfile
FROM node:8
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . /usr/src/app
RUN npm install
EXPOSE 443
CMD [ "npm", "start" ]
```

Now when we do `npm run deploy` and query our server, the queries should show up in [our Engine UI](https://engine.apollographql.com/).
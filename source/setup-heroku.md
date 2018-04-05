---
title: Running a Server and Engine on Heroku
sidebar_title: Node with Engine on Heroku
description: Get your Node.js GraphQL server with Engine deployed on Heroku
---

This section describes how to deploy your GraphQL service with production-ready Engine features. Before starting, make sure you have [added Engine to your GraphQL server](setup-node.html). If you haven't built a GraphQL server yet, check out [our tutorial](https://dev-blog.apollodata.com/tutorial-building-a-graphql-server-cddaa023c035) first and then come back when you're ready to deploy!

<h3 id="add-engine-key" title="Add Engine Key">1. Add `process.env.ENGINE_API_KEY` to the Engine constructor</h3>

This way, Engine will get your API key from an environment variable, which we'll configure in the Heroku dashboard in the next step.

```js
const engine = new ApolloEngine({
  apiKey: process.env.ENGINE_API_KEY
});
```

The API key represents your service's secret ID in Engine. To get one, log into the [Engine UI](https://engine.apollographql.com) and create a service.

<h3 id="configure-heroku" title="Configure Heroku">2. Create and set up a new Heroku application</h3>

Log into the [Heroku dashboard](https://dashboard.heroku.com/apps). Then click “New” > “Create New App” in the top right

<div style="text-align:center">
![New App Screenshot](img/setup-heroku/new-app.png)
<br></br>
</div>

Name your app and hit “Create app”

<div style="text-align:center">
![Create App Screenshot](img/setup-heroku/create-app.png)
<br></br>
</div>

Select a deployment method. GitHub is a common choice, which requires selecting a repository:

<div style="text-align:center">
![Add Integration Screenshot](img/setup-heroku/add-integration.png)
<br></br>
</div>

Put you Apollo Engine API key in the environment. Under the “Settings” tab, click “Reveal Config Vars". Next copy your key from the [Engine UI](http://engine.apollographql.com/) as the value for ENGINE_API_KEY.

<div style="text-align:center">
![Add Engine Api Key Screenshot](img/setup-heroku/add-engine-key.png)
<br></br>
</div>

<h3 id="test-and-add" title="Test and Expand">3. Try it out and build more!</h3>

Send a query to your GraphQL service at your Heroku Application at `<APP NAME>.herokuapp.com` and then check out the tracing data in the [Engine UI](http://engine.apollographql.com/).

To get the most out of Engine, read on about some of the features:

- [Performance tracing](./performance.html) will help you learn how your GraphQL execution is working.
- [Caching](./caching.html) will enable you to reduce load on your server and reduce response times.

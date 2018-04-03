---
title: Running a Server and Engine on Heroku
sidebar_title: Node with Engine on Heroku
description: Get your Node.js GraphQL server with Engine deployed on Heroku
---

This section describes how to deploy your GraphQL service that is made production ready by Engine. Before starting, make sure you have enabled your GraphQL server with Engine, which you can complete in [three steps](setup-node.html). If you are brand new, you should complete [this tutorial](https://dev-blog.apollodata.com/tutorial-building-a-graphql-server-cddaa023c035) first and then come back when you're ready to deploy!

<h3 id="add-engine-key" title="Add Engine Key">1. Add `process.env.ENGINE_API_KEY` to the Engine constructor</h3>

Engine will look for the API key in the process's environment, which we will setup next.

```js
const engine = new ApolloEngine({
  apiKey: process.env.ENGINE_API_KEY
});
```

> An API key represents your services unique id and can be generated on the [Engine UI](https://engine.apollographql.com).

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

Send a query to your GraphQL service at your Heroku Application at `\<APP NAME\>.herokuapp.com/` and then checkout the tracing data in the [Engine UI](http://engine.apollographql.com/). Now check out the [features](./performance.html) to learn how GraphQL execution data is used or if you haven't already, you can add [caching](./caching.html)!

<div style="text-align:center">
<iframe src="https://www.graphqlbin.com" width="100%" height="400"></iframe>
</div>


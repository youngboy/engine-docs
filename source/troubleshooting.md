---
title: Engine Troubleshooting
order: 3
---

<h2 id="sanity-checks" title="Sanity checks">Sanity checks</h2>

We've found that the following steps can help iron out many errors. Let's go through each level of setting up Engine properly. If you are still having an issue, you may have a unique use case or have found a bug that we'd like to hear about! Email us at [support@apollodata.com](mailto:support@apollodata.com).

Is your server is one of the supported GraphQL servers listed [here](http://engine-docs.apollographql.com/docs/engine/index.html#apollo-tracing)?

If so, please upgrade to the latest released versions of GraphQL Server, Engine and npm packages.
You can enter the following into the commandline to check the package version, or look in  `package.json`. 
```
$ npm view <apollo-engine or npm package> version
0.4.10
```
**Supported package versions**

graphql-js 0.10+
apollo-engine 0.4.11+
apollo-tracing 0.9+

<h3 id="check-config" title="Check configuration">Check your Engine configuration</h3>

Try a `diff` between the relevant configuration options:

**Node**
```
{
  engineConfig: {
    apiKey: engineApiKey,
    logcfg: {
      level: 'DEBUG'   // Engine Proxy logging level. DEBUG, INFO, WARN or ERROR
    }
  },
  graphqlPort: process.env.PORT || 8003,  // GraphQL port
  endpoint: '/graphql',                   // GraphQL endpoint suffix - '/graphql' by default
  dumpTraffic: true                       // Debug configuration that logs traffic between Proxy and GraphQL server
}
```
**Ruby, Java, Elixir, Scala**
```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  },
  "origins": [
    {
      "url": "http://localhost:3000/graphql"
    }
  ],
  "frontends": [
    {
      "host": "127.0.0.1",
      "port": 3001,
      "endpoint": "/graphql"
    }
  ]
}
```
**AWS Lambda**
```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  },
  "origins": [
    {
      "url":"arn:aws:lambda:xxxxxxxxxxx:xxxxxxxxxxxx:function:xxxxxxxxxxxxxxxxxxx",
      "awsAccessKeyId":"xxxxxxxxxxxxxxxxxxxx",
      "awsSecretAccessKey":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "originType": "Lambda"
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

<h4 id="up-logging-level" title="Up Logging Level">Up the logging level in your config file</h2>

Set the Engine Proxy logging level to DEBUG or higher. These logs will be part of your GraphQL server logs (if Proxy is deployed as a sidecar) or in the Proxy process logs (if Proxy is deployed standalone).
```
const engine = new Engine({
  engineConfig: {
    apiKey: engineApiKey,
    logcfg: {
      level: 'DEBUG'   // Engine Proxy logging level. DEBUG, INFO, WARN or ERROR
    }
  },
  graphqlPort: process.env.PORT || 8003,  
  endpoint: '/graphql',                  
  dumpTraffic: true                       
});
```
<h3 id="validate-each-step" title="Validate Each Step">Let's go through the installation once more, validating each step as correct.</h2>

<h4 id="test=tracing" title="Test Apollo Tracing">Did you integrate Apollo Tracing?</h3>

Test it! if your GraphQL server returns trace extensions in GraphQL responses, it's is a sign that Apollo Tracing is properly configured.

(If using sidecar) Did you integrate the Engine middleware into your server? Is it called Before all other middleware calls?

<h4 id="start-container" title="Start Docker Container">(If using standalone) Start the standalone Docker proxy container.</h3>

If you are still seeing the issue, please email [support@apollodata.com](mailto:support@apollodata.com). 

<h2 id="troubleshooting-faq" title="Troubleshooting FAQ">Troubleshooting FAQ</h2>

**I'm getting an error saying “The query failed!”, how do I fix it?**

This may mean you need to upgrade an NPM package. Check that your package versions are all up-to-date. This also may mean a variety of other things. When this error is paired with a 503 error, the query did not receive an expected response.

Other common errors:

Query error: HTTP Status Code 400
```
Error parsing JSON query!\njson: cannot unmarshal array into Go value of type graphql.SerializedQuery\n[{\"query\":\"query 	
```
This is a known issue. As of version 0.4.11, Engine does not support batched queries. If you would like an update when the fix is released, please contact [support@apollodata.com](mailto:support@apollodata.com). 

**Why isn't data showing up in my dashboard? I followed your install guide exactly.**

We'd recommend double checking that your Engine API key is passing through to your server. 

Second, check the ordering of your middleware calls. One of the most common reasons is that the application middleware actually begins before the Engine proxy middleware, which needs to be called before. This is an easy step to miss!

Third, we currently support a certain range of language-specific GraphQL servers and middleware implementations in Node.js: Express, Hapi, Koa, Restify, and Lambda. If you are using a middleware like Restify, the proxy may not be supported. 

**Why do my logs show urls like frontend, origin, and passthrough urls?**

Each time the Engine proxy starts, you should see the following 4 lines in the logs indicating the Engine proxy is healthy: 

``` 
time="2017-10-16T14:34:48-07:00" level=info msg="Creating frontend" endpoint=/graphql host=127.0.0.1 port=62590 
time="2017-10-16T14:34:48-07:00" level=info msg="Starting HTTP frontend" frontend="127.0.0.1:62590/graphql" 
time="2017-10-16T14:34:48-07:00" level=info msg="Added new origin" url="http://127.0.0.1:3010/graphql" (http://127.0.0.1:3010/graphql); 
time="2017-10-16T14:34:48-07:00" level=info msg="Set passthrough url" url="http://127.0.0.1:3010” (http://127.0.0.1:3010/); 
``` 

These lines are internal to the Engine proxy: the endpoint url that you want will be the same as you configured your application to run without Engine. 

**How do I find my GraphiQL endpoint on my application server?**

Visit the domain in the browser where you are running your application. 

such as http://localhost:3010 (http://localhot:3010/) or https://a-startup.com (https://mydomain.com/) + `/graphiql`. 

**Will Engine show *all* errors that my application throws?**

Engine only logs errors that occur as part of a GraphQL request. That would include things like throw new Error() in a resolver, but not in parts of your code that are not hit when serving a request.

**I'm still having issues...**

Please include the following when submitting an issue to our support team:

* Engine JSON configuration object
* Type of GraphQL server and whether you are using a sidecar or standalone container
* The query submitted and the full response

Submit your issue to [support@apollodata.com](mailto:support@apollodata.com) or join our Apollo public slack channel here (https://apollographql.slack.com/).


---
title: Setup Troubleshooting
order: 3
---

If you hit any issues in setting up Engine for your GraphQL service, we're here to help! First, follow these troubleshooting steps to check for any obvious issues. If these don't help, please submit a support ticket to [support@apollodata.com](mailto:support@apollodata.com) and we'll work with you to get you up and running!

<h2 id="sanity-checks" title="First steps">First Troubleshooting steps</h2>

<h3> Check that you are on a supported GraphQL server </h3>

Check that your server is one of the supported GraphQL servers listed [here](index.html#apollo-tracing).

If it is, please make sure you're running the [currently tested version](https://github.com/apollographql/apollo-engine-js/blob/master/package.json) of Apollo Server and your Node HTTP server package (Express, Connect, Hapi, Koa, etc), and the latest released versions of the Engine and Apollo packages.

You can enter the following into the commandline to check the package version, or look in  `package.json`.
```
$ npm view apollo-engine version
0.4.10
```

**Supported package versions**

graphql-js 0.10+
apollo-engine 0.4.11+
apollo-tracing 0.0.9+

<h3> Check for Engine configuration validity </h3>

These are sample configurations for Engine based on the server environment you are using. A comprehensive documentation on the configurations are available [here](proto-doc.html)  Use these as a guide to validate your configuration.

**Node sidecar**
```
{
  engineConfig: {
    apiKey: engineApiKey,
    logging: {
      level: 'DEBUG'   // Engine Proxy logging level. DEBUG, INFO, WARN or ERROR
    }
  },
  graphqlPort: process.env.PORT || 8003,  // GraphQL port
  endpoint: '/graphql',                   // GraphQL endpoint suffix - '/graphql' by default
  dumpTraffic: true                       // Debug configuration that logs traffic between Proxy and GraphQL server
}
```
**Ruby, Java, Elixir, Scala, or Node with Docker container**
```
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
**AWS Lambda**
```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  },
  "origins": [
    {
      "lambda": {
        "url":"arn:aws:lambda:xxxxxxxxxxx:xxxxxxxxxxxx:function:xxxxxxxxxxxxxxxxxxx",
        "awsAccessKeyId":"xxxxxxxxxxxxxxxxxxxx",
        "awsSecretAccessKey":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "originType": "Lambda"
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

<h3> Set debug logging levels for the Proxy</h3>

Set the Engine Proxy logging level to DEBUG or higher. These logs will be part of your GraphQL server logs (if Proxy is deployed as a sidecar) or in the Proxy process logs (if Proxy is deployed standalone).
```
const engine = new Engine({
  engineConfig: {
    apiKey: engineApiKey,
    logging: {
      level: 'DEBUG'   // Engine Proxy logging level. DEBUG, INFO, WARN or ERROR
    }
  },
  graphqlPort: process.env.PORT || 8003,  
  endpoint: '/graphql',                  
  dumpTraffic: true                       
});
```

<h3> Ensure you enabled Apollo Tracing </h3>

Test that you enaled Apollo Tracing by checking if your GraphQL server returns trace extensions in GraphQL responses. If it does, it's is a sign that Apollo Tracing is properly configured.

If using the sidecar deployment configuration - check that you integrated the Engine middleware into your server. Is it called Before all other middleware calls?

<h2 id="">Troubleshooting FAQs</h2>

**I'm receiving an error. What does this mean?**

| Error message | Issue description and resolution steps |
| 503 error - unable to communicate with backend | This likely means that the origin GraphQL server did not respond to the proxied request. To resolve this, verify the Origin specification in the Engine configuration matches the corresponding GraphQL server location |
| 403 error - [location unavailable] | Received unsupported Content-Type from origin GraphQL server. This often occurs when a GraphQL server throws an error during resolver execution and sends that error as `text/plain` instead of embedding it within a GraphQL error. This could also be caused by an inaccurate Content-Type header. |

**Where is the data in my dashboard? I followed all installation steps and am sending successful queries.**

We'd recommend double checking that your Engine API key is passing through to your server. 

Second, check the ordering of your middleware calls. One of the most common reasons is that the application middleware actually begins before the Engine proxy middleware, which needs to be called before. This is an easy step to miss!

Third, we currently support a certain range of language-specific GraphQL servers and middleware implementations in Node.js: Express, Hapi, Koa, Restify, and Lambda. If you are using a middleware like Restify, the proxy may not be supported. 

**What is shown on the Engine Proxy logs?**

Each time the Engine proxy starts, you should see the following lines in the logs indicating the Engine proxy is healthy: 

``` 
{ proxy:
   { level: 'info',
     logLevel: 'debug',
     msg: 'Set log level.',
     time: '2018-03-01T12:33:35-08:00' } }
{ proxy:
   { level: 'debug',
     msg: 'Created origin.',
     name: '',
     time: '2018-03-01T12:33:35-08:00',
     type: 'http',
     url: 'http://127.0.0.1:3010/graphql' } }
{ proxy:
   { host: '127.0.0.1',
     level: 'debug',
     msg: 'Creating frontend service.',
     port: 0,
     time: '2018-03-01T12:33:35-08:00' } }
{ proxy:
   { address: '127.0.0.1:64201',
     level: 'info',
     msg: 'Started HTTP server.',
     time: '2018-03-01T12:33:35-08:00' } }
{ proxy:
   { configured: 3,
     current: 0,
     level: 'debug',
     msg: 'Synchronizing stores.',
     time: '2018-03-01T12:33:35-08:00' } }
{ proxy:
   { level: 'debug',
     msg: 'Refreshing store configuration.',
     storeName: 'publicResponseCache',
     time: '2018-03-01T12:33:35-08:00',
     type: 'embedded' } }
{ proxy:
   { level: 'debug',
     msg: 'Refreshing store configuration.',
     storeName: 'privateResponseCache',
     time: '2018-03-01T12:33:35-08:00',
     type: 'embedded' } }
{ proxy:
   { level: 'debug',
     msg: 'Refreshing store configuration.',
     storeName: 'pq',
     time: '2018-03-01T12:33:35-08:00',
     type: 'embedded' } }
{ proxy:
   { level: 'info',
     msg: 'Engine proxy started.',
     time: '2018-03-01T12:33:35-08:00',
     version: '2018.02-37-g678cbb68b' } }
``` 

These lines are internal to the Engine proxy: the endpoint url that you want will be the same as you configured your application to run without Engine. 

<h2 id="">Submit a support ticket</h2> 

Please include the following when submitting an issue to our support team:

* Engine JSON configuration object
* Type of GraphQL server and whether you are using the `apollo-engine` sidecar or standalone Docker container
* The query submitted and the full response, including headers

Submit your issue to [support@apollodata.com](mailto:support@apollodata.com) or join our Apollo public slack #engine channel here (https://apollographql.slack.com/).

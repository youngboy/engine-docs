---
title: Setup for PaaS Hosting Environments (e.g. Heroku)
sidebar_title: PaaS (e.g. Heroku)
description: Get Engine running with your virtually hosted GraphQL server.
order: 2
---

To get started with Engine, you will need to:
1. Instrument your server with a supported tracing agent that uses the Apollo Tracing format.
2. Configure and deploy the Engine proxy docker container.
3. Send requests to your service – you're all set up!

> NOTE: If you're using Node, we recommend running the Engine proxy as a sidecar with the npm package as opposed to within a Docker container inside of a separate app / dyno. For sidecar setup, follow the instructions [here](setup-node.html).  For all other platforms, the only available option is to run the proxy in a standalone docker container.

_Interested in writing a sidecar for your platform? [Get in touch](mailto:support@apollodata.com) with us!_

<h2 id="enable-apollo-tracing" title="Enable Apollo Tracing">1. Enable Apollo Tracing</h2>
You will need to instrument your GraphQL server with a [tracing package](apollo-tracing.html) that matches your server platform.  Engine relies on receiving data in this format to create its performance telemetry reports.

<h2 id="configure-proxy" title="Configure the Proxy">2. Configure the Proxy</h2>
<h3 id="get-api-key" title="Get your API Key">2.1 Get your API Key</h3>
First, get your Engine API key by creating a service on http://engine.apollographql.com/. You will need to log in and click "Add Service" to recieve your API key.

<h3 id="create-config-json" title="Create your Config.json">2.2 Follow the [Engine Heroku Example](https://github.com/apollographql/engine-heroku-example) to configure your Proxy</h3>
The standalone proxy uses a JSON file placed in the Docker container's root folder to get configuration information. Changes to the config.json will cause the proxy to adopt the new configuration without downtime.  

> NOTE: For Virtual Hosted environments where the `PORT` is dynamic, you can follow the guide [here](https://github.com/apollographql/engine-heroku-example) for specific instructions on how to set the port dynamically.

**Create a JSON configuration file:**

```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logging": {
    "level": "INFO"
  },
  "origins": [
    {
      "http": {
        "url": "http://yourappname.herokuapp.com/graphql",
        "overrideRequestHeaders": {
          "Host": "yourappname.herokuapp.com"
        }
      }
    }
  ],
  "frontends": [
    {
      "host": "0.0.0.0",
      "port": ${PORT}, // You may need to supply this at runtime as per example mentioned above.
      "endpoint": "/graphql"
    }
  ]
}
```

**Configuration options:**
1. `apiKey`: The API key for the Engine service you want to report data to.
2. `logging.level` : Logging level for the proxy. Supported values are `DEBUG`, `INFO`, `WARN`, `ERROR`.
3. `origin.http.url` : The URL for your GraphQL server.
4. `origins.http.overrideRequestHeaders.Host`: set to the origin hostname so virtual hosting systems can properly route to the origin.
5. `frontend.host` : The hostname the proxy should be available on.
6. `frontend.port` : The port the proxy should bind to.
7. `frontend.endpoint` : The path for the proxy's GraphQL server . This is usually `/graphql`.

For full configuration details see [Proxy config](proto-doc.html).

It does not matter where you choose to deploy and manage your Engine proxy. We run our own on Amazon's [EC2 Container Service](https://aws.amazon.com/ecs/).

We recognize that almost every team using Engine has a slightly different deployment environment, and encourage you to [contact us](mailto: support@apollodata.com) with feedback or for help if you encounter problems running the Engine proxy.

<h2 id="view-metrics-in-engine" title="View Metrics in Engine">3. View Metrics in Engine</h2>
Once your server is set up, navigate your new Engine service on https://engine.apollographql.com. Start sending requests to your GraphQL server to start seeing performance metrics!


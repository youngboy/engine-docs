---
title: Configure a Standalone Docker Container
order: 20
---

This option is available to you if you want to run the Engine proxy process separately from your GraphQL server or if there is no language-specific Engine sidecar for your server. These instructions are for setting up a Docker container that will run the entire Engine proxy process.

<h2 id="get-api-key" title="Get your API Key">Get your Engine API Key</h2>
First, get an Engine API Key by creating a service on http://engine.apollographql.com/. You will need to log in and click "Add Service" to recieve your API key.

<h2 id="create-config-json" title="Create your Config.json">Create your Proxy's Config.json</h2>
The proxy uses a JSON object to get configuration information. If the configuration is passed the path to your file, that file will be watched for changes. Changes will cause the proxy to adopt the new configuration without downtime.

Create a JSON configuration file:

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

What are those things?
1. `origin.http.url` : The URL for your GraphQL server
2. `frontend.host` : The hostname the proxy should be available on
3. `frontend.port` : The port the proxy should bind to
4. `frontend.endpoint` : The path for the GraphQL server . This is usually /graphql.
5. `logging.level` : Logging level for the proxy. Supported values are DEBUG, INFO, WARN, ERROR .

For full configuration details see [Proxy config](proto-doc.html).

<h2 id="run-the-proxy" title="Run the Proxy">Run the proxy Docker container:</h2>

```
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine:2017.11-121-g2a0310e1b
```


<h2 id="view-metrics-in-engine" title="View Metrics in Engine">Next step!</h2>
Once you've gotten your docker container up and running, return to the language-specific instructions for your server to continue by setting up tracing.

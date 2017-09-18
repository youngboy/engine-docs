---
title: Configure Engine for Lambda Node servers
order: 2
---

These are the set of steps you'll follow to configure Engine for Lambda Node servers:

* Configure and deploy the Engine proxy Docker container
* Install Apollo tracing for your GraphQL Node server code running on Lambda
* Deploy your server - you are then all set up!

## Configure and deploy the Engine proxy Docker container

Get an API Key by creating a service on http://engine.apollographql.com/.

The proxy uses a JSON object to get configuration information. If the configuration is provided as a filename, that file will be watched for changes. Changes will cause the proxy to adopt the new configuration without downtime.

Create a JSON configuration file:

```
{
  "origins": [
    {
      "url":"arn:aws:lambda:xxxxxxxxxxx:xxxxxxxxxxxx:function:xxxxxxxxxxxxxxxxxxx",
      "id":"xxxxxxxxxxxxxxxxxxxx",
      "secret":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "originType": "Lambda"
    }
  ],
  "frontends": [
    {
      "host": "127.0.0.1",
      "port": 3001,
      "endpoint": "/graphql"
    }
  ],
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  }
}
```
Where:

1. frontend.host : The hostname the proxy should be available on
2. frontend.port : The port the proxy should bind to
3. frontend.endpoint : The path for the GraphQL server . This is usually /graphql.

Run the proxy Docker container:

```
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine-ea-confidential:2017.08-54-gaae01ac9
```

## Install Apollo Tracing

Install Apollo tracing in your GraphQL server code that runs on Lambda to enable Engine to receive performance traces for your GraphQL requests.

Use the Node.js tracing package, with instructions here: https://github.com/apollographql/apollo-tracing-js

## View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!


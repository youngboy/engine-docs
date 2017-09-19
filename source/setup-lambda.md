---
title: Setup for Lambda Node Servers
order: 2
---

To get started with Engine for Lambda Node servers, take the following steps:

1. Configure and deploy the Engine proxy docker container
2. Install Apollo tracing for your GraphQL Node server code running on Lambda
3. Deploy your server - you're all set up!

## 1. Install the Proxy

### Get your Engine API Key
First, get an Engine API Key by creating a service on http://engine.apollographql.com/. You will need to log in and click "Add Service" to recieve your API key.

### Create your Proxy's Config.json
The proxy uses a JSON object to get configuration information. If the configuration is passed the path to your file, that file will be watched for changes. Changes will cause the proxy to adopt the new configuration without downtime.

Create a JSON configuration file:

```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  },
  "origins": [
    {
      "url":"arn:aws:lambda:xxxxxxxxxxx:xxxxxxxxxxxx:function:xxxxxxxxxxxxxxxxxxx",
      "id":"xxxxxxxxxxxxxxxxxxxx",
      "secret":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "originType": "Lambda"
    }
  ],"origins": [
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

What are those things?
1. `frontend.host` : The hostname the proxy should be available on
2. `frontend.port` : The port the proxy should bind to
3. `frontend.endpoint` : The path for the GraphQL server . This is usually /graphql.

### Run the proxy Docker container:

```
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine-ea-confidential:2017.09-17-g4679f9a4
```

## 2. Instrument Apollo Tracing

Install Apollo tracing in your GraphQL server code that runs on Lambda to enable Engine to receive performance traces for your GraphQL requests.

Use the Node.js tracing package, with instructions here: https://github.com/apollographql/apollo-tracing-js

## 3. View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!


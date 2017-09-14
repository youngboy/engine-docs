---
title: Configure Engine for Scala Servers
order: 2
---

These are the set of steps you'll follow to configure Engine:

* Configure and deploy the Engine proxy Docker container
* Install Apollo tracing for your GraphQL server
* Deploy your server - you are then all set up!

## Configure and deploy the Engine proxy Docker container

Get an API Key by creating a service on http://engine.apollographql.com/.

```
{
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
  ],
  "apiKey": "<ENGINE_API_KEY>",
  "logcfg": {
    "level": "INFO"
  }
}
```

1. origin.URL : The URL for your GraphQL server
2. frontend.host : The hostname the proxy should be available on
3. frontend.port : The port the proxy should bind to
4. frontend.endpoint : The path for the GraphQL server . This is usually /graphql.

To run the proxy:

```
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine-ea-confidential:2017.08-19-g8eb44893
```

## Install Apollo Tracing

Install Apollo tracing in your GraphQL server to enable Engine to receive performance traces for your GraphQL requests.

Use the Scala tracing package, with instructions here: https://gist.github.com/OlegIlyenko/124b55e58609ad45fcec276f15158d16

## View Metrics in Engine

Once your server is set up, navigate to the endpoint in your Engine account - https://engine.apollographql.com. You'll then be able to see performance metrics!
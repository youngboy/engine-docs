---
title: Setup for AWS Lambda
order: 2
---

To get started with Engine for AWS Lambda, you will need to:
1. Instrument your function with a tracing agent that uses the Apollo Tracing format.
2. Configure and deploy the Engine proxy docker container.
3. Send requests to your service – you're all set up!

The Engine proxy will invoke the Lambda function as if it was called from [API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html#api-gateway-simple-proxy-for-lambda-input-format), and the function should return a value suitable for [API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html#api-gateway-simple-proxy-for-lambda-output-format).

We suggest using NodeJS, but any runtime supported by Lambda can be used.

**Supported Node servers:** [Apollo Server](https://github.com/apollographql/apollo-server) (Express, Hapi, Koa, Restify, and Lambda); [Express-GraphQL](https://github.com/graphql/express-graphql)


## 1. Instrument Function with Apollo Tracing

You will need to instrument your Lambda function with a tracing package that follows the [Apollo Tracing](https://github.com/apollographql/apollo-tracing) format. Engine relies on receiving data in this format to create its performance telemetry reports.

For NodeJS, this is our recommended npm package: https://github.com/apollographql/apollo-tracing-js

## 2. Configure the Proxy

The only available option for running the Engine proxy with a function on Lambda is to run the proxy in a standalone docker container. The Proxy is required as it is responsible for capturing, aggregating and then sending to Engine the trace data from each Lambda instance GraphQL response.

### 2.1 Get your API Key
First, get your `Engine_API_Key` by creating a service on https://engine.apollographql.com/. You will need to log in and click "Add Service" to recieve your API key.

### 2.2 Create your Proxy's Config.json
The proxy uses a JSON object to get configuration information. If the configuration is passed the path to your file, that file will be watched for changes. Changes will cause the proxy to adopt the new configuration without downtime.

**Create a JSON configuration file:**

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

**Configuration options:**
1. `apiKey` : The API key for the Engine service you want to report data to.
2. `logcfg.level` : Logging level for the proxy. Supported values are `DEBUG`, `INFO`, `WARN`, `ERROR`.
3. `origin.url` : The Lambda function to invoke, in the form:
                  arn:aws:lambda:xxxxxxxxxxx:xxxxxxxxxxxx:function:xxxxxxxxxxxxxxxxxxx
4. `origin.awsAccessKeyId` : Your Access Key ID. If not provided the proxy will attempt `AWS_ACCESS_KEY_ID`/`AWS_SECRET_KEY` environment variables, and EC2 instance profile.
5. `origin.awsSecretAccessKey` : Your Secret Access Key.
6. `origin.originType` : Set to `Lambda` to specify a Lambda GraphQL server origin.
7. `frontend.host` : The hostname the proxy should be available on. For Docker, this should always be `0.0.0.0`.
8. `frontend.port` : The port the proxy should bind to.
9. `frontend.endpoint` : The path for the proxy's GraphQL server . This is usually `/graphql`.

For full configuration details see [Proxy config](/proto-doc.html).

### 2.3 Run the Proxy (Docker Container)

The Engine proxy is a docker image that you will deploy and manage separate from your server.

If you have a working [docker installation](https://docs.docker.com/engine/installation/), type the following lines in your shell (variables replaced with the correct values for your environment) to run the Engine proxy:
```
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine:2017.10-19-gdc8304d2
```

This will make the Engine proxy available at `http://localhost:3001`.

It does not matter where you choose to deploy and manage your Engine proxy. We run our own on Amazon's [EC2 Container Service](https://aws.amazon.com/ecs/).

We recognize that almost every team using Engine has a slightly different deployment environment, and encourage you to [contact us](mailto: support@apollodata.com) with feedback or for help if you encounter problems running the Engine proxy.

## 3. View Metrics in Engine

Once your server is set up, navigate your new Engine service on https://engine.apollographql.com. Start sending requests to your Node server to start seeing performance metrics!

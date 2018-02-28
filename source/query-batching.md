---
title: Query batching
sidebar_title: Query Batching 
description: Query batching recommendations for Engine
---

Query batching allows your client to batch multiple queries into one request.  This means that if you render several view components within a short time interval, for example a navbar, sidebar, and content, and each of those do their own GraphQL query, the queries can be sent together in a single roundtrip.  

> Note that we don't recommend enabling query batching with Engine for most uses. Engine supports HTTP/2 even when your server doesn't, and one of the features that allows is sending multiple requests in parallel over a single connection. In most cases, that is actually more efficient than query batching, because requests are not blocked on each other. If you want to use HTTP/2, make sure to use the sidecar in single proxy mode or deploy Engine using the Docker container.

A batch of queries can be sent by simply sending a JSON-encoded array of queries in the request:

```js
[ 
 { "query": "{
  feed(limit: 2, type: NEW) {
    postedBy {
      login
    }
    repository {
      name
      owner {
        login
      }
    }
  }
}" }, 
 { "query": "query CurrentUserForLayout {
  currentUser {
    __typename
    avatar_url
    login
  }
}" }
] 
```

Batched requests to servers that don’t support batching fail without explicit code to handle batching, however Engine ships with batched request handling built-in.  

<h2 id="apollo-server-batch-support" title="Batching, Apollo Client & Engine">Batching and Engine</h2>

If a batch of queries is sent, the batches are fractured by the Engine proxy and individual queries are sent to origins in parallel.  Engine will wait for all the responses to complete and send a single response back to the client.  The response will be an array of GraphQL results:

```js
[{
  "data": {
    "feed": [
      {
        "postedBy": {
          "login": "AleksandraKaminska"
        },
        "repository": {
          "name": "GitHubApp",
          "owner": {
            "login": "AleksandraKaminska"
          }
        }
      },
      {
        "postedBy": {
          "login": "ashokhein"
        },
        "repository": {
          "name": "memeryde",
          "owner": {
            "login": "ashokhein"
          }
        }
      }
    ]
  }
},
{
  "data": {
    "currentUser": {
      "__typename": "User",
      "avatar_url": "https://avatars2.githubusercontent.com/u/11861843?v=4",
      "login": "johannakate"
    }
  }
}]
```

<h2 id="apollo-server-batch-support" title="Batching, Apollo Client & Engine">Batching in Apollo Client with Engine</h2>

Apollo Client has built-in support for batching queries in your client application.  To learn how to use query batching with Apollo Client, visit the in-depth guide on our package [`apollo-link-batch-http`](https://www.apollographql.com/docs/link/links/batch-http.html).


<h2 id="apollo-server-batch-support" title="Batching, Apollo Server & Engine">Batchin in Apollo Server with Engine</h2>

If your origin supports batching and you'd like to pass entire batches through, set "supportsBatch": true within the origins section of the configuration:

```js
{
  "apiKey": "<ENGINE_API_KEY>",
  "logging": {
    "level": "DEBUG"
  },
  "origins": [
    {
      "http": {
        "url": "http://localhost:3000/graphql"
      },
      "supportsBatch": true
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

Add this setting, and you're good to go! 

If you have questions, we're always available in the public [Apollo Slack #engine channel](https://www.apollographql.com/#slack).



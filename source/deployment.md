---
title: Deploy your GraphQL server with Engine
sidebar_title: Deploy
description: Deploy your GraphQL server with Engine
---

This guide is a walk-through for deploying your GraphQL server  instrumented
with Engine to a variety of popular deployment platforms. These guides generally
focus on deploying **Node** GraphQL servers. For non-Node servers, you will need
to deploy Engine separately from your server. For details on how to do that,
see the guide for [non-Node servers](./setup-standalone.html).

For most platforms, you won't need to change any details of your deployment
process, but for some, there are a few "gotchas" to look out for. We've put
together a couple of guides for the platforms we know require making some
changes. If a platform you're using isn't listed and your typical deployment
pattern isn't working, 
[reach out to us](https://www.apollographql.com/support) so we can help 
sort it out and add it to this listing!

<h3 id="Guides">Deployment Guides</h3>

At the moment, we have deployment guides for:  
[Heroku with Node](./deploy-heroku.html)  
[Heroku with non-Node](./setup-virtual.html)  
[Zeit Now](./deploy-now.html)  
[Azure](./deploy-azure.html)

<h3 id="heroku">Common "Gotchas"</h3>
We recognize that every deployment platform imposes different constraints,
so the best way to verify that your deployment process is unaffected by
Engine is to understand the [constraints that Engine imposes](./index.html)
and compare those to the constraints of the platform you're deploying to. 
These are a few of the most common constraints we've seen to affect users.

<h4 id="host-header">GOTCHA #1: The Host Header</h4>
Some virtual environments rely on the `Host` request header in order to 
route traffic. Because Engine operates as a proxy to your app, there are
two different hosts that are communicating with your underlying app: the
address of your app and the address of Engineproxy. By default, Engine will
forward the Host header sent in the original request to it to your GraphQL
server. However, if the virtual-hosted environment uses the `Host` header
for routing traffic **and you're hosting Engine separately from your 
GraphQL server**, it may try to route back to Engineproxy, resulting in
an infinite loop. Note that this "gotcha" only affects standalone Engine
deployments.
<br></br>  
To resolve this, you can use `overrideRequestHeaders` in
the `Origin.HTTP` 
[proxy configuration](https://www.apollographql.com/docs/engine/proxy-config.html#Origin-HTTP)
to set the `Host` header to be overridden to the address of your GraphQL server.
To see an example of a deployment process that hits this "gotcha", take a look
at the [standalone Heroku deployment guide](./setup-virtual.html).

<h4 id="multiple-ports">GOTCHA #2: Multiple Ports</h4>
Many virtual environments will only allow you to expose _one_ port for your 
deployment. That constraint in and of itself is completely fine, but problems
can occur depending on _how_ the platform determines the port to expose. Because
Engine acts as a proxy to your GraphQL server, the 
[`engine.listen(...)`](https://www.apollographql.com/docs/engine/setup-node.html#apollo-engine)
call is actually setting up _two_ ports to listen on: The port that Engine will listen
on to route traffic, and the port that your GraphQL server will listen on. In the
case of [Zeit Now Node deployments](https://zeit.co/docs/deployment-types/node#port-selection),
this results in ambiguously specifying which port your deployment should expose. To
skirt this issue, see [the Now deployment guide](./deploy-now.html) provided by a
community member, [@lorensr](https://github.com/lorensr). 
 
<h4 id="iisnode">GOTCHA #3: IISNode</h4>
As mentioned above, Engine will, by default, listen on the `port` passed into the
call to `engine.listen(...)` to route traffic to your GraphQL server. If the
platform you're using is hosted on Windows machines using 
[named pipes](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365590.aspx),
as is the case with [Azure](https://azure.microsoft.com/overview) 
(and any deployment using IISNode), you will need to specifically listen on a 
`pipePath` instead of a `port`. For a more detailed walkthrough using Azure, see
[the Azure deployment guide](./deploy-azure.html).
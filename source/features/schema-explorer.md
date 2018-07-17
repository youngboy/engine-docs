---
title: Schema explorer
description: The single source of truth for your GraphQL schema
---

Your GraphQL schema is the life blood of your application, but schemas are often hard to keep track of—especially at high development velocity. With Engine's powerful schema registry, there is always one source of truth for your schema, and Engine's _Schema Explorer_ is the place to see it. This view in Engine is the hub for your development team and provides the information product developers need when building features.

<h2 id="schema-content">Type List</h2>

Most schemas are complex webs of concepts, making it easy for even experienced developers to get lost in their schema and difficult for new developers to learn a new schema. The Schema Explorer displays a list of all the types and fields in your schema, along with their doc strings, so developers can quickly explore your schema. Related types are hyperlinked for fast navigation.

![Type List](../img/schema-explorer/explorer.png)

<h2 id="schema-content">Field Usage</h2>

When you [send usage metrics](./setup-apollo-server.html), you will also see usage stats on each field in your schema. _Total Requests_ gives you a sense of how in demand a field was in the past hour, while _Total Duration_ (requests multiplied by service time) lets you know how expensive the field is. Sort by either one of these fields to focus on the types/fields that matter most to the performance of your application.

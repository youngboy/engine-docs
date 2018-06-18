---
title: Schema History
description: Safely evolve your schema over time
---

GraphQL makes evolving your API much easier than it used to be with REST. You can add and remove types, fields, and arguments as needed to serve the changing needs of your clients, without breaking the previous consumers of the API.

In order to do this safely, it is critical to know how clients are actually using the schema. With schema history in Apollo Engine, you can evolve your schema while using real world usage to validate your changes. You can also see how your schema has changed over time, and track changes back to specific commits.

For more information and best practices about versioning your GraphQL endpoint over time, check out our in-depth [guide](https://www.apollographql.com/docs/guides/versioning.html).

XXX screenshot

<h2 id="setup">Publishing a schema</h2>

To get started with schema history in Engine, you need to publish the current version of your schema. Here's how:

<h3 id="install-apollo-cli">Install Apollo CLI</h3>

Schemas are published to Engine using the Apollo command line interface. To use this, you can either install the CLI globally, or as a development dependency. To install the Apollo CLI globally, run the following command in your terminal:

```bash
npm i -g apollo
```

Note that you'll need to have `node` and `npm` installed on your machine.

<h3 id="publish-schema">Publish schema</h3>

Once you have the Apollo CLI installed, the next step is to publish your schema. The CLI can read your schema from one of three sources:

1. A running GraphQL endpoint that has introspection enabled
2. A file with the introspection query result
3. A file with the schema in the GraphQL schema definition language

The quickest way to publish the current version of your schema is to run the following command, pointed at your GraphQL server:

```bash
apollo schema:publish --service="<ENGINE_API_KEY>" --endpoint="https://example.com/graphql"
```

When running this command, replace the `<ENGINE_API_KEY>` with the api key from your service in Engine, and replace the url with the location of your GraphQL endpoint.

<h2 id="history">Version History</h2>

As your schema grows and evolves to meet the needs of your product, it is helpful to see a history of changes for a team. This allows everyone to know when new features were introduced, when old fields were removed, and even link back to the commit that caused the change. Engine provides all the tooling needed to track this history in a simple way. Everytime your schema is updated, you can simply run the [publish](#publish-schema) command again to keep an up to date history of your schema.

XXX screenshot

<h2 id="schema-validation">Schema Validation</h2>

GraphQL schemas can change in a number of ways between releases. Some of these changes are safe for clients, some could lead to unexpected edge cases, and some will immediately break current usage.

To safely evolve your schema, it is critical to know what is different between the new version and the current schema, and how the API is being used. Apollo Engine provides detailed and powerful schema diffing backed by usage information to provide a safe and robust way to evolve your schema.

To check and see the difference between the current published schema and a new version, run the following command:

```bash
apollo schema:check --service="<ENGINE_API_KEY>" --endpoint="http://localhost:4000/graphql"
```

When running this command, replace the `<ENGINE_API_KEY>` with the api key from your service in Engine, and replace the url with the location of a GraphQL endpoint running the new schema.

Engine will report back three types of changes:

1. **Failure:** either the schema is not valid, or current client usage will break with these changes
2. **Warning:** there are potential problems that may come from this change, but no clients are immediately impacted
3. **Notice:** this change is safe to make and will not break any current usage

The more usage information that Engine has through [reporting performance metrics](./performance.html), the better the report of these changes will become.

XXX screenshot

<h2 id="github">Github Integration</h2>

Schema validation is best used when integrated with your teams development workflow. To make this easy, Apollo Engine integrates with Github to provide status checks on pull requests when you make schema changes. To enable schema validation in Github, follow these steps:

<h3 id="install-github">Install Github Application</h3>

Go to [https://github.com/apps/apollo-engine](https://github.com/apps/apollo-engine) and click the `Configure` button to install the Engine integration on the Github profile or organization of your choice.

<h3 id="check-schema-on-ci">Run Validation on Commit</h3>

Within your CI (continuous integration such as Circle CI) environment, you will want to run the [schema validation](#schema-validation) command on every branch. This will report the proposed schema changes back to GitHub, and you'll be able to see them as part of that PR's status check info.

To run the validation command, you will need to run your server to enable introspection, and then run the `schema:check` command. Below is a sample configuration for Circle CI:

```yaml
version: 2

jobs:
  build:
    docker:
      - image: circleci/node:8

    steps:
      - checkout

      - run: npm install
      - run: npm i -g apollo

      # npm start will start a server running at http://localhost:4000/graphql
      # this is the default for apollo-server 2.0
      - run:
          name: Running Server
          command: npm start
          background: true

      # this assumes that ENGINE_API_KEY is set in the environment variables
      - run: apollo schema:check

      # this command will publish the latest version of the schema
      # to Engine but only on the master branch
      - run:
          command: |
            if [ "${CIRCLE_BRANCH}" == "master" ]; then
              apollo schema:publish
            fi
```


XXX screenshot

<h2 id="cli-commands">CLI Usage</h2>

* [`apollo help [COMMAND]`](#apollo-help-command)
* [`apollo schema:check`](#apollo-schemacheck)
* [`apollo schema:publish`](#apollo-schemapublish)

<h3 id="apollo-help-command">`apollo help [COMMAND]`</h3>

Display help for the Apollo CLI:

```
USAGE
  $ apollo help [COMMAND]

ARGUMENTS
  COMMAND  command to show help for

OPTIONS
  --all  see all commands in CLI
```

<h3 id="apollo-schemacheck">`apollo schema:check`</h3>

Check a schema against previous published schema.

```
USAGE
  $ apollo schema:check

OPTIONS
  -e, --endpoint=endpoint  [default: http://localhost:4000/graphql] The location of the server to from which to fetch
                           the schema

  -h, --help               show CLI help

  -s, --service=service    ENGINE_API_KEY for the Engine service

  --header=header          Additional headers to send to server for introspectionQuery

  --json                   output result as json
```

<h3 id="apollo-schemapublish">`apollo schema:publish`</h3>

Publish a schema to Engine.

```
USAGE
  $ apollo schema:publish

OPTIONS
  -e, --endpoint=endpoint  [default: http://localhost:4000/graphql] The location of the server to from which to fetch
                           the schema

  -h, --help               show CLI help

  -s, --service=service    ENGINE_API_KEY for the Engine service

  --header=header          Additional headers to send to server for introspectionQuery

  --json                   output successful publish result as json
```


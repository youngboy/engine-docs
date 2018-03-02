---
title: Apollo Engine Security Features
sidebar_title: Security Features
description: Learn how Apollo Engine works to secure your data
---

<h2 id="security-practices">Engine Security Practices</h2>

The following describes practices for working with your application's sensitive data and Engine's reporting service. This information and guide is not comprehensive. If you have further questions about how Apollo Engine provides a secure GraphQL proxy service, please email [Apollo Security](security@apollographql.com).

<h3 id="security-overview" title="Security Overview">Security Features</h3>

Apollo Engine is a proxy service that acts as a GraphQL gateway, capable of caching your GraphQL queries, tracing performance metrics, tracking errors, and more. Because Engine reports and aggregates your GraphQL query metrics and errors, there are a few actions you can take to prevent variable names or reports to be sent to the [Engine Service](https://engine.apollographql.com).

<h4 id="variable-blacklisting" title="Variable Blacklisting">Variable Blacklisting</h4>
Added the ability to prevent certain GraphQL variables, by name, from being forwarded to Apollo Engine servers and reported to the [Engine Service UI](https://engine.apollographql.com). The proxy replaces these variables with the string (redacted) in traces, so their presence can be verified but the value is not transmitted.

To blacklist GraphQL variables password and secret, add: "privateVariables": ["password", "secret"] within the reporting section of the configuration. There are no default private variables.

<h4 id="disable-reporting" title="Disable Reporting">Disable Reporting</h4>

We've also added the option to disable reporting of stats and traces to Apollo servers, so that integration tests can run without polluting production data.

To disable reporting, add "disabled": true within the reporting section of the configuration. Reporting is enabled by default.

<h3 id="policies" title="Policies and Agreements">Policies and Agreements</h2>

[Terms of Service](https://d14xs1qewsqjcd.cloudfront.net/assets/content/Meteor-Terms-of-Service.pdf)
[Privacy Policy](https://d14xs1qewsqjcd.cloudfront.net/assets/content/Meteor-Privacy-Policy.pdf)
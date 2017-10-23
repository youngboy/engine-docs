---
title: Proxy release notes
order: 20
---

### 2017.10-376-g0e29d5d5

* Added (debug) log message to indicate if a query's trace was selected for reporting.
* Fixed an issue where non-GraphQL errors (i.e. a `500` response with an HTML error page) would not be tracked as errors.

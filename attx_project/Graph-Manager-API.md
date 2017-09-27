# Graph Manager API

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
- [Graph Manager API](#graph-manager-api)
    - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Building Graph Manager Service](#building-graph-manager-service)
    - [Graph Store Connection](#graph-store-connection)
  - [Environment Variables](#environment-variables)
    - [health Endpoint](#health-endpoint)
    - [graph Endpoint](#graph-endpoint)
  - [Graph Manager with Message Broker](#graph-manager-with-message-broker)

<!-- TOC END -->



## Overview

The Graph Manager REST API has the following endpoints:
* `graph` - managing interaction with the Graph Store and retrieving statistics about it (e.g. list of named graphs, number of queries);
* `health` - checks if the application is running.

The Graph Manager Services

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4302/version/endpoint`)

## Building Graph Manager Service

For formatted the GM-API see:
* [GraphManager Service source](https://github.com/ATTX-project/graphmanager-service) - building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Gradle).

### Graph Store Connection

The GM API requires a connection with the [Graph Store](Graph-Store.md), such a connection is specified either in the indexing or clustering or linkage requests.
A graph store connection is required:
* perform operations on the graph data stored such as update and replace;
* query and retrieve graph data, in order to place data in the distribution component.

## Environment Variables

* `MHOST` - container name or address for the MYSQL database (defaults to localhost);
* `MUSER` - user name;
* `MPROVQUEUE` - password or key for that user;
* `MRPCQUEUE` - password or key for that user;
* `MKEY` - table which to access (defaults to `unified_views`).
* `GHOST` - (defaults to localhost);
* `GPORT` -
* `GKEY` -
* `DS` - .

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4302/health reponds with `200` in order for other applications to know the service is running.

### graph Endpoint

* `graph/query`
* `graph/update`
* `graph?uri={graphURI}`
* `graph/list`
* `graph/statistics`

## Graph Manager with Message Broker

Graph Manager can work with the [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to update the provenance in the Graph Store.

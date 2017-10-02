# Graph Manager API

### Table of Contents


## Overview

The Graph Manager REST API has the following endpoints:
* `graph` - managing interaction with the Graph Store and retrieving statistics about it (e.g. list of named graphs, number of queries);
* `health` - checks if the application is running.

The Graph Manager Services

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4302/version/endpoint`)

## Building Graph Manager Service

For formatted the GM-API see:
* [GraphManager Service source](https://github.com/ATTX-project/graphmanager-service) - building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Building-with-Gradle.md).

### Graph Store Connection

The GM API requires a connection with the [Graph Store](Graph-Store.md), such a connection is specified either in the indexing or clustering or linkage requests.
A graph store connection is required:
* perform operations on the graph data stored such as update and replace;
* query and retrieve graph data, in order to place data in the distribution component.

### Message Broker Connection

Graph Manager can work with the [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to manage the data in the Graph Store.

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to localhost);
* `MUSER` - user name for MessageBroker;
* `MPROVQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MRPCQUEUE` - RPC Queue for responding to messages received by the GraphManager Queue (defaults to `attx.graphManager.inbox`);
* `MKEY` - password for MessageBroker;
* `GHOST` - Graph Store container name or address (defaults to localhost);
* `GPORT` - Graph Store port;
* `GKEY` - Graph Store password;
* `DS` - Graph Store working dataset.

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4302/health reponds with `200` in order for other applications to know the service is running.

### graph Endpoint

* `graph/query`
* `graph/update`
* `graph?uri={graphURI}`
* `graph/list`
* `graph/statistics`

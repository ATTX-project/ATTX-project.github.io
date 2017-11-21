<h1 style="color:red">Work In Progress</h1>

# Graph Manager API

Github repository: https://github.com/ATTX-project/graphmanager-service

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
  - [Overview](#overview)
  - [Building Graph Manager Service](#building-graph-manager-service)
    - [Graph Store Connection](#graph-store-connection)
    - [Message Broker Connection](#message-broker-connection)
  - [Environment Variables](#environment-variables)
  - [Service API Endpoints](#service-api-endpoints)
    - [health Endpoint](#health-endpoint)
    - [graph Endpoint](#graph-endpoint)

<!-- TOC END -->

## Overview

The Graph Manager REST API has the following endpoints:
* `graph` - managing interaction with the Graph Store and retrieving statistics about it (e.g. list of named graphs, number of queries);
* `health` - checks if the application is running.

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4302/version/endpoint`)

## Building Graph Manager Service

In order to work with the service start with:
* [GraphManager Service source](https://github.com/ATTX-project/graphmanager-service) - contains information on building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Building-with-Gradle.md).

### Graph Store Connection

The Graph Manager API requires a connection with the [Graph Store](Graph-Store.md).
A graph store connection is required in order to:
* perform operations on the graph data stored such as update and replace;
* query and retrieve graph data.

### Message Broker Connection

Graph Manager can work with the [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order facilitate other services to manage the data in the Graph Store.

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MPROVQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MRPCQUEUE` - RPC Queue for responding to messages received by the GraphManager Queue (defaults to `attx.graphManager.inbox`);
* `MKEY` - password for MessageBroker (defaults to `password`);
* `GHOST` - Graph Store container name or address (defaults to `localhost`);
* `GPORT` - Graph Store port (defaults to `3030`);
* `GKEY` - Graph Store password (defaults to `pw123`);
* `DS` - Graph Store working dataset (defaults to `ds`)
* `DATADIR` - data directory for storing results from graph store (defaults to `/attx-sb-shared`).

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4302/health reponds with `200` in order for other applications to know the service is running.

### graph Endpoint

* `graph/query` POST API
    * Request:
    ```json
    {
      "targetGraph": ["default"],
      "query": "SELECT ?subject ?predicate ?object WHERE { ?subject ?predicate ?object} LIMIT 25",
      "contentType": "application/sparql-results+xml"
    }
    ```
* `graph/update` POST API
    * Request:
    ```json
    {
      "targetGraph": "http://data.hulib.helsinki.fi/attx/strategy",
      "triples": "<http://example/egbook3> <http://purl.org/dc/elements/1.1/title>  \"This is an example title\".",
      "contentType": "text/turtle"
    }
    ```
* `graph/construct` POST API
* `graph?uri={graphURI}` GET API
* `graph/list` GET API
* `graph/statistics` GET API

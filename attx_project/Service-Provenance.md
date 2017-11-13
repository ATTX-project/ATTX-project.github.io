<h1 style="color:red">Work In Progress</h1>

# Provenance Service

Github repository: https://github.com/ATTX-project/provenance-service

Considering a Microservices oriented system we identify two means of introducing a Provenance system in such a system:
* **Explicit Provenance** Making all services of that system provenance aware - meaning each service will have to send messages to the Provenance Services that contain provenance related information;
* **Implicit Provenance** Enable the system to capture in the communication between distinct services provenance related information and distribute it to the Provenance Service.

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
  - [Overview](#overview)
  - [Building Provenance Service](#building-provenance-service)
    - [Graph Store Connection](#graph-store-connection)
    - [Message Broker Connection](#message-broker-connection)
  - [Environment Variables](#environment-variables)
  - [Service API Endpoints](#service-api-endpoints)
    - [health Endpoint](#health-endpoint)
    - [graph Endpoint](#graph-endpoint)
    - [prov Endpoint](#prov-endpoint)
  - [Request/response documents](#requestresponse-documents)

<!-- TOC END -->

## Overview

The Provenance Service REST API has the following endpoints:
* `graph` - managing interaction with the Graph Store and retrieving statistics about it (e.g. list of named graphs, number of queries);
* `prov` - endpoint for registering provenance information directly;
* `status` - status of a provenance job with a specific ID;
* `health` - checks if the application is running.

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:7030/version/endpoint`)

## Building Provenance Service

In order to work with the service start with:
* [Provenance Service source](https://github.com/ATTX-project/provenance-service) - contains information on building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Building-with-Gradle.md).

### Graph Store Connection

The Provenance service requires a connection with the [Graph Store](Graph-Store.md).
A graph store connection is required in order to:
* perform operations on the graph data stored such as update and replace;
* query and retrieve graph data.

### Message Broker Connection

Provenance Service needs [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to communicate with other services that want to register provenance information in the Graph Store.

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MKEY` - password for MessageBroker (defaults to `password`);
* `GHOST` - Graph Store container name or address (defaults to `localhost`);
* `GPORT` - Graph Store port (defaults to `3030`);
* `GKEY` - Graph Store password (defaults to `pw123`);
* `DS` - Graph Store working dataset (defaults to `ds`).

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:7030/health reponds with `200` in order for other applications to know the service is running.

### graph Endpoint

* `graph/query`
    * Request:
    ```json
    {
      "namedGraph": "default",
      "query": "SELECT ?subject ?predicate ?object WHERE { ?subject ?predicate ?object} LIMIT 25"
    }

    ```
* `graph/update` POST API:
    * Request:
    ```json
    {
      "namedGraph": "http://data.hulib.helsinki.fi/attx/strategy",
      "triples": "<http://example/egbook3> <http://purl.org/dc/elements/1.1/title>  \"This is an example title\"."
    }
    ```
* `graph?uri={graphURI}`
* `graph/list` GET API
* `graph/statistics` GET API

### prov Endpoint

* `prov` POST API - register provenance message. The response is a status of a task if the message was processed along with a task ID
* `status/task/{task_id}` GET API - retrieved the processed provenance with a given task ID

## Request/response documents

See [Provenance examples](Examples-Provenance-Service.md)

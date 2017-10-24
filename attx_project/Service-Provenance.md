<h1 style="color:red">Work In Progress</h1>

# Provenance Service

Github repository: https://github.com/ATTX-project/provenance-service

Considering a Microservices oriented system we identify two means of introducing a Provenance system in such a system:
* **Explicit Provenance** Making all services of that system provenance aware - meaning each service will have to send messages to the Provenance Services that contain provenance related information;
* **Implicit Provenance** Enable the system to capture in the communication between distinct services provenance related information and distribute it to the Provenance Service.

### Table of Contents


## Overview

The Graph Manager REST API has the following endpoints:
* `graph` - managing interaction with the Graph Store and retrieving statistics about it (e.g. list of named graphs, number of queries);
* `prov` - endpoint for registering provenance information directly;
* `status` - status of a provenance job with a specific ID;
* `health` - checks if the application is running.

The Provenance Services

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:7030/version/endpoint`)

## Building Graph Manager Service

For formatted the GM-API see:
* [Provenance Service source](https://github.com/ATTX-project/graphmanager-service) - building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Building-with-Gradle.md).

### Graph Store Connection

The GM API requires a connection with the [Graph Store](Graph-Store.md), such a connection is specified either in the indexing or clustering or linkage requests.
A graph store connection is required:
* perform operations on the graph data stored such as update and replace;
* query and retrieve graph data, in order to place data in the distribution component.

### Message Broker Connection

Provenance Service needs [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to update the provenance in the Graph Store.

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to localhost);
* `MUSER` - user name for MessageBroker;
* `MQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MKEY` - password for MessageBroker;
* `GHOST` - Graph Store container name or address (defaults to localhost);
* `GPORT` - Graph Store port;
* `GKEY` - Graph Store password;
* `DS` - Graph Store working dataset.

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:7030/health reponds with `200` in order for other applications to know the service is running.

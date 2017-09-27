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

### Graph Store Connection

The GM API requires a connection with the [Graph Store](Graph-Store.md), such a connection is specified either in the indexing or clustering or linkage requests.
A graph store connection is required:
* perform operations on the graph data stored such as update and replace;
* query and retrieve graph data, in order to place data in the distribution component.

## Environment Variables

* `MHOST` - container name or address for the MYSQL database (defaults to localhost);
* `MUSER` - user name;
* `MQUEUE` - password or key for that user;
* `MKEY` - table which to access (defaults to `unified_views`).
* `GHOST` - (defaults to localhost);
* `GPORT` -
* `GKEY` -
* `DS` - .

## Provenance Service with Message Broker

Graph Manager can work with the [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to update the provenance in the Graph Store.

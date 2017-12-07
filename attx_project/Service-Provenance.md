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
* `index` - index provenance information in Elasticsearch;
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

### Elasticsearch Connection

In order to index provenance data into Elasticsearch, we require both the [Graph Framing Service](Service-Graph-Framing.md) and the [Indexing Service](Service-Indexing.md). The parameters for these services as as follows:
* alias: `attx`;
* document type: name graph in Graph Store e.g. if the named graph is `http://data.hulib.helsinki.fi/prov_workflow1_activity6` the document type is `workflow1_activity6`.

Provenance JSON-LD frame:
```json
{
  "@type": "http://www.w3.org/ns/prov#Activity",
  "http://www.w3.org/ns/prov#qualifiedAssociation": {
    "@explicit": "true",
    "http://www.w3.org/ns/prov#hadPlan": {
      "@explicit": true,
      "http://www.w3.org/2000/01/rdf-schema#label": ""
    }
  },
  "http://www.w3.org/ns/prov#qualifiedCommunication": {
    "@explicit": "true",
    "http://www.w3.org/ns/prov#activity": ""
  },
  "http://www.w3.org/ns/prov#qualifiedGeneration": {
    "@explicit": "true",
    "http://www.w3.org/ns/prov#entity": ""
  },
  "http://www.w3.org/ns/prov#qualifiedUsage": {
    "@explicit": "true",
    "http://www.w3.org/ns/prov#entity": ""
  }
}
```

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `FRAMEQUEUE`- Graph Framing queue in the MessageBroker (defaults to `attx.ldframe.inbox`);
* `INDEXQUEUE`- Elasticsearch Indexing queue in the MessageBroker (defaults to `attx.indexing.inbox`);
* `MKEY` - password for MessageBroker (defaults to `password`);
* `GHOST` - Graph Store container name or address (defaults to `localhost`);
* `GPORT` - Graph Store port (defaults to `3030`);
* `GKEY` - Graph Store password (defaults to `pw123`);
* `DS` - Graph Store working dataset (defaults to `ds`);
* `FRAMEHOST` - container name for Graph Framing Service (defaults to `localhost`);
* `FRAMEVER` - version of the Graph Framing Service API (defaults to `0.2`);
* `FRAMEPORT` - port for the Graph Framing Service (defaults to `4303`);
* `INDEXHOST`- container name for the Elasticsearch Indexing Service (defaults to `localhost`);
* `INDEXVER` - version of the Elasticsearch Indexing Service API (defaults to `0.2`);
* `INDEXPORT` - port for the Elasticsearch Indexing Service (defaults to `4304`);
* `QTIME` - variable to set up the timer for publisher (defaults to 9 minutes and input must be integer).

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

# Ontology Service

Github repository: https://github.com/ATTX-project/ontology-service

The service makes use of: https://jena.apache.org/documentation/inference

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
  - [Overview](#overview)
  - [Building Ontology Service](#building-ontology-service)
    - [Message Broker Connection](#message-broker-connection)
  - [Environment Variables](#environment-variables)
  - [Service API Endpoints](#service-api-endpoints)
    - [health Endpoint](#health-endpoint)
    - [infer Endpoint](#infer-endpoint)
    - [report Endpoint](#report-endpoint)

<!-- TOC END -->

## Overview

The Ontology REST API has the following endpoints:
* `infer` - infer new data based on a schema/ontlogy and a data graph;
* `report` - validated RDF data agains a schema/ontology and retrieve a report;
* `health` - checks if the application is running.

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4305/version/endpoint`)

## Building Ontology Service

In order to work with the service start with:
* [Ontology Service source](https://github.com/ATTX-project/ontology-service) - contains information on building both the project and the  Docker image and testing from the source code;

Build with Gradle and run:

```
gradle run
```

### Message Broker Connection

Ontology Service needs [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) .

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MPASS` - password for MessageBroker (defaults to `password`).

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4305/health reponds with `200` in order for other applications to know the service is running.

### alias Endpoint

* `/infer` POST API: infer new data.

```
{
  "schemaGraph": "file:///resources/owlDemoSchema.ttl",
  "dataGraph":"file:///resources/owlDemoData.ttl"
}
```

### data Endpoint

* `/report` POST API: validity report.

```
{
  "schemaGraph": "file:///resources/owlDemoSchema.ttl",
  "dataGraph":"file:///resources/owlDemoData.ttl"
}
```

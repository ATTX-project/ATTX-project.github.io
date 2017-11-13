<h1 style="color:red">Work In Progress</h1>

# GraphFraming Service

Github repository: https://github.com/ATTX-project/graphframing-service

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
  - [Overview](#overview)
  - [Building Graph Framing Service](#building-graph-framing-service)
    - [Graph Manager Service Connection](#graph-manager-service-connection)
    - [Message Broker Connection](#message-broker-connection)
  - [Environment Variables](#environment-variables)
  - [Service API Endpoints](#service-api-endpoints)
    - [health Endpoint](#health-endpoint)

<!-- TOC END -->

## Overview

The Graph Framing REST API has the following endpoints:
* `health` - checks if the application is running.

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4303/version/endpoint`)

## Building Graph Framing Service

In order to work with the service start with:
* [Graph Framing Service source](https://github.com/ATTX-project/graphframing-service) - contains information on building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Building-with-Gradle.md).

### Graph Manager Service Connection

The Graph Framing Service requires a connection with [Graph Manager Service](Service-Graph-Manager.md) such a connection is required to retrieve graph with a given URI from the Graph Store, graphs that will be used along side an JSON-LD frame.

### Message Broker Connection

Provenance Service needs [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order for the communication with UV plugins.

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MPROVQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MRPCQUEUE` - RPC Queue for responding to messages received by the GraphManager Queue (defaults to `attx.ldframe.inbox`);
* `GMHOST` - GraphManager container name or address (defaults to `localhost`);
* `GMPORT` - GraphManager port (defaults to `4302`).

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4303/health reponds with `200` in order for other applications to know the service is running.

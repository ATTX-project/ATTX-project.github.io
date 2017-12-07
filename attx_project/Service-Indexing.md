# Indexing Service

Github repository: https://github.com/ATTX-project/indexing-service

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
  - [Overview](#overview)
  - [Building Indexing Service](#building-indexing-service)
    - [Elasticsearch Connection](#elasticsearch-connection)
    - [Message Broker Connection](#message-broker-connection)
  - [Environment Variables](#environment-variables)
  - [Service API Endpoints](#service-api-endpoints)
    - [health Endpoint](#health-endpoint)
    - [alias Endpoint](#alias-endpoint)
    - [data Endpoint](#data-endpoint)

<!-- TOC END -->

## Overview

The Indexing REST API has the following endpoints:
* `alias` - information on aliases from Elasticsearch;
* `data` - index data to Elasticsearch;
* `health` - checks if the application is running.

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4304/version/endpoint`)

## Building Indexing Service

In order to work with the service start with:
* [Indexing Service source](https://github.com/ATTX-project/indexing-service) - contains information on building both the project and the  Docker image and testing from the source code;
* short tutorial on [Gradle](Building-with-Gradle.md).

### Elasticsearch Connection

The Indexing Service requires a connection with [Elasticsearch](ElasticSearch-5.md).
The connection with Elasticsearch is required  in order to:
* perform operations such as bulk or simple index and replace index data with a given alias;
* retrieve aliases data, in order to index data or replace deprecated indexes.

### Message Broker Connection

Provenance Service needs [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to communicate with UV plugins.

## Environment Variables

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MPROVQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MRPCQUEUE` - RPC Queue for responding to messages received by the GraphManager Queue (defaults to `attx.indexing.inbox`);
* `MKEY` - password for MessageBroker (defaults to `password`);
* `ESHOST` - Elasticsearch container name or address (defaults to `localhost`);
* `ESPORT` - Elasticsearch port (defaults to `9210`);
* `ESPORTB` - Elasticsearch second port (defaults to `9310`).

## Service API Endpoints

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4304/health reponds with `200` in order for other applications to know the service is running.

### alias Endpoint

* `alias/list` GET API: retrieve a list of aliases from Elasticsearch.

### data Endpoint

* `data/index` POST API: register data in Elasticsearch using the message format.

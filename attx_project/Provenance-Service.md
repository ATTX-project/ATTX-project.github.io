# Provenance Service

Github repository: https://github.com/ATTX-project/provenance-service

Considering a Microservices oriented system we identify two means of introducing a Provenance system in such a system:
* **Explicit Provenance** Making all services of that system provenance aware - meaning each service will have to send messages to the Provenance Services that contain provenance related information;
* **Implicit Provenance** Enable the system to capture in the communication between distinct services provenance related information and distribute it to the Provenance Service.

### Table of Contents


## Overview

The Graph Manager REST API has the following endpoints:
* `graph`
* `prov`
* `health` - checks if the application is running.

The Graph Manager Services

Current version: `0.1` (URL for the endpoint should take into consideration for the API `http://host:7030/version/endpoint`)


## Provenance Service with Message Broker

Graph Manager can work with the [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to update the provenance in the Graph Store.

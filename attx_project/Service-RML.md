<h1 style="color:red">Work In Progress</h1>

# RML Service

Github repository: https://github.com/ATTX-project/rml-service



RML service is implemented as a SpringBoot application. It uses spring-boot-starter-amqp package to communicate with the RabbitMQ messagebroker, spring-boot-starter-web package to implement the application specific REST API and spring-boot-starter-actuator package to implement the introspection related APIs.

Service can receive transformation request documents through REST api and RabbitMQ request queue.

## Endpoints

* GET /health - returns status of the service. Will return "not available", if the connection to the message broker is not available, otherwise returns status=up.
* GET /metrics - Spring actuator's basic implementation.
* POST /version/transform

Current version is: 0.1

## Building and running RML service

Build with Gradle and run as Spring boot application:

```
gradle bootRun
```

## Environment Variables (required)

* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (default='amqp://localhost')
* `MEXCHANGE` - exchange to use (default='')
* `MQUEUE` - request queue in the MessageBroker (default:'rmlservice')
* `MUSER` - user name for MessageBroker (default='user')
* `MPASS` - password for MessageBroker (default='password')

## RML Mapping

Logical source must always use variable ```{filename}```
Example:

```
  rml:logicalSource [
    rml:source "{filename}"  ;
    rml:referenceFormulation ql:JSONPath ;
    rml:iterator "$.[*]"
  ];
```

## Request/response documents

See [examples](Examples-RML-Service.md)

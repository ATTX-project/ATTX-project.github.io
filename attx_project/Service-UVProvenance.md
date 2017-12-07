# UVProvenance API

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Overview](#overview)
  - [Building the UVProvenance-API](#building-the-uvprovenance-api)
    - [Database Connection](#database-connection)
  - [Consuming the UVProvenance-API](#consuming-the-uvprovenance-api)
    - [health Endpoint](#health-endpoint)
    - [provjob Endpoint](#provjob-endpoint)
    - [workflow Endpoint](#workflow-endpoint)
    - [activity Endpoint](#activity-endpoint)
  - [UVProvenance with Messaging Queues](#uvprovenance-with-messaging-queues)
  - [How the Information is Extracted](#how-the-information-is-extracted)

<!-- TOC END -->

## Overview

The UV (UnifiedViews) Provenance REST API has three endpoints along with a health check endpoint:
* `workflow` - retrieve all the workflows and associated steps from the ETL Artifact;
* `activity` - retrieve all successfully completed activities and associated datasets from the ETL Artifact;
* `provjob` - start the provenance update job without waiting for the scheduled run;
* `health` - checks if the application is running.

The general purpose of the UV Provenance API is to review metadata for workflows and activities such that it exposes information that is according with the [Ontology/Data model](ATTX-Data-Model.md). Such information could not easily be obtained directly from the [UnifiedViews REST API](https://grips.semantic-web.at/display/UDDOC/UnifiedViews+REST+API). Moreover the configuration necessary to extract this information is encoded in the Pipeline DPU configuration, as depicted in examples below.

Current version: `0.2` (URL for the endpoint should take into consideration for the API `http://host:4301/version/endpoint`).

The UV Provenance application also has a Message-based communication part, which implements a basic message Producer that will send provenance messages as described in the [Provenance Service](Provenance-Service.md) to a specific queue (in our case `provenance.inbox`) in order to update the (missing) provenance information in the Graph Store.

## Building the UVProvenance-API

For building the UVProvenance-API see:
* [UVProvenance-API source](https://github.com/ATTX-project/uv-provenance-service) - repository
* Docker Image build - https://github.com/ATTX-project/platform-deployment/tree/dev/service-uvprov
* short tutorial on [Gradle](Building-with-Gradle.md)

### Database Connection

In order to run the tests successfully but also the UVProvenance-API successfully one would require an existing MYSQL database connection on which the UnifiedViews runs. Such a connection can be configured in Docker file via the environment variables:
* `MHOST` - container name or address for the MessageBroker-RabbitMQ database (defaults to `localhost`);
* `MUSER` - user name for MessageBroker (defaults to `user`);
* `MQUEUE` - provenance queue in the MessageBroker (defaults to `provenance.inbox`);
* `MKEY` - password for MessageBroker (defaults to `password`);
* `DBHOST` - container name or address for the MYSQL database (defaults to localhost);
* `DBUSER` - database user name;
* `DBKEY` - password or key for the database user;
* `DBASE` - table which to access (defaults to `unified_views`);
* `QTIME` - variable to set up the timer for publisher (defaults to 9 minutes and input must be integer).

At the same time it can be troubleshoot using the docker container:
* connect to the running UVProvenance-API Docker container `docker exec -it container-id /bin/sh` (replace `container-id` with the ID of the docker container)
* run python
* in the python shell:
  * `import pymysql as mysql`
  * `mysql.connect(host='addressOfDBServer', port=3306, user='username', passwd='password', db='database', charset='utf8')` - replace the variables in `''` with the correct information. Running this command with help with debugging mysql connection issues.

## Consuming the UVProvenance-API

As specified above the UV Provenance API has two endpoints:
* `workflow` - retrieve all the workflows and associated steps from the ETL Artifact (only GET method possible);
* `activity` - retrieve all successfully completed activities and associated datasets from the ETL Artifact (only GET method possible).
* `provjob` - start the provenance update job without waiting for the scheduled run (only GET method possible) - responds with `202`;
* `health` - responds with `200` so that we know the service is alive.

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4301/health responds with `200` in order for other applications to know the service is running.

### provjob Endpoint
  * `provjob` GET API:
    * simple: http://localhost:4301/provjob responds with `202` in order to start the provenance update job without waiting for the scheduled run.

### workflow Endpoint

* `workflow` GET API:
  * simple: http://localhost:4301/0.2/workflow
  * Responses: if status codes are `304` No Modified or `405` Method Not Allowed the response body is empty. If the Response status is `200` it will return data specific to workflows:

  ```json
    [
      {
       "provenance": {
           "context": {
               "activityID": "3",
               "stepID": "2",
               "workflowID": "2"
           },
           "agent": {
               "role": "ETL",
               "ID": "UnifiedViews"
           },
           "activity": {
               "status": "success",
               "type": "StepExecution",
               "title": "uv-dpu-l-attx-uploader"
           }
       },
       "payload": {}
    },
    {
       "provenance": {
           "context": {
               "activityID": "4",
               "stepID": "3",
               "workflowID": "3"
           },
           "agent": {
               "role": "ETL",
               "ID": "UnifiedViews"
           },
           "activity": {
               "status": "failed",
               "type": "StepExecution",
               "title": "uv-t-sparqlConstruct"
           }
       },
       "payload": {}
    }
    ]
    ```
### activity Endpoint

* `activity` GET API:
  * simple: http://localhost:4301/0.2/activity
  * Responses: if status codes are `304` No Modified or `405` Method Not Allowed the response body is empty. If the Response status is `200` it will return data specific to activites:

  ```json
  [
  	{
  		"provenance": {
  			"context": {
  				"activityID": "2",
  				"workflowID": "1"
  			},
  			"agent": {
  				"role": "ETL",
  				"ID": "UnifiedViews"
  			},
  			"activity": {
  				"status": "warning",
  				"endTime": "2017-09-06T06:51:03",
  				"type": "WorkflowExecution",
  				"startTime": "2017-09-06T06:51:03",
  				"title": "name Activity2"
  			}
  		},
  		"payload": {}
  	},
  	{
  		"provenance": {
  			"context": {
  				"activityID": "3",
  				"workflowID": "2"
  			},
  			"agent": {
  				"role": "ETL",
  				"ID": "UnifiedViews"
  			},
  			"activity": {
  				"status": "success",
  				"endTime": "2017-09-06T11:39:12",
  				"type": "WorkflowExecution",
  				"startTime": "2017-09-06T11:39:12",
  				"title": "ceva Activity3"
  			}
  		},
  		"payload": {}
  	},
  	{
  		"provenance": {
  			"context": {
  				"activityID": "4",
  				"workflowID": "3"
  			},
  			"agent": {
  				"role": "ETL",
  				"ID": "UnifiedViews"
  			},
  			"activity": {
  				"status": "failed",
  				"endTime": "2017-09-06T11:40:16",
  				"type": "WorkflowExecution",
  				"startTime": "2017-09-06T11:40:16",
  				"title": "error pipeline Activity4"
  			}
  		},
  		"payload": {}
  	}
  ]

  ```
* `activity` and `workflow` POST APIs will return `405` as this operation current is not allowed.

## UVProvenance with Message Broker

The UVProvenance update can work with the [RabbitMQ Message Broker](MessageBroker-RabbitMQ.md) in order to update the provenance in the Graph Store. It will send the message to the `provenance.inbox` queue every 30 minutes.
By default it processes all activities and worfklows, in order to update the provenance information if it changed - e.g. an activity was run with new parameters and its status changed from `failed` to `success`.

## How the Information is Extracted

Extracting information associated to the provenance and workflow information can be classified in three types:
* Database information related to pipelines and pipeline execution.

Information specific to workflows and activities is extracted directly from the MySQL database for example activities and associated information are extracted from pipeline executions.

```sql
SELECT exec_pipeline.id AS 'activityId',
ppl_model.id AS 'workflowId',
exec_pipeline.t_start AS 'activityStart',
exec_pipeline.t_end AS 'activityEnd',
exec_pipeline.t_end AS 'lastChange'
FROM exec_pipeline, ppl_model
WHERE exec_pipeline.pipeline_id = ppl_model.id AND\
exec_pipeline.status = 5
ORDER BY ppl_model.id
```

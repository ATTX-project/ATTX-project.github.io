<h1 style="color:red">TO BE REFACTORED</h1>

# Graph Manager API

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Overview](#overview)
  - [Building the GM-API](#building-the-gm-api)
    - [Graph Store Connection](#graph-store-connection)
    - [Elasticsearch Connection](#elasticsearch-connection)
    - [UVProvovenance API Connection](#uvprovovenance-api-connection)
  - [Consuming the GM-API](#consuming-the-gm-api)
    - [health Endpoint](#health-endpoint)
    - [index Endpoint](#index-endpoint)
    - [cluster Endpoint](#cluster-endpoint)
    - [link Endpoint](#link-endpoint)
    - [linkstrategy Endpoint](#linkstrategy-endpoint)
    - [prov Endpoint](#prov-endpoint)
  - [Processing and Indexing Graph Data](#processing-and-indexing-graph-data)
    - [Elasticsearch 5.x and JSON derived from JSON-LD frame](#elasticsearch-5x-and-json-derived-from-json-ld-frame)
    - [Elasticsearch 5.x indexing JSON-LD](#elasticsearch-5x-indexing-json-ld)
    - [Elasticsearch 1.3.4 and Siren plugin](#elasticsearch-134-and-siren-plugin)

<!-- TOC END -->



## Overview

The Graph Manager REST API has the following endpoints:
* `index` - on given indexing parameters index the data from the Graph Store and insert it into Distribution Component endpoint;
* `cluster` - cluster the IDs from the working data in the Graph Store;
* `link` - retrieve the links in the Graph Store see: [Linking Graphs](Linking-graphs.md);
* `linkstrategy` - retrieve either all or specific linking strategies from the Graph Store;
* `prov` - retrieve provenance from the UVProvovenance API and update the Graph Store with the provenance information.
* `health` - checks if the application is running.

The GM exposes information from the Graph Store to the Distribution component (Elasticsearch). It runs the indexing processing, clustering of IDs for the data and also it communicates with the UVProvovenance API about the Provenance information in the Graph Store.

Current version: `0.1` (URL for the endpoint should take into consideration for the API `http://host:4302/version/endpoint`)

## Building the GM-API

For formatted the GM-API see:
* [GM-API source](https://github.com/ATTX-project/graph-component/blob/dev/gm-API/README.md) - building both the project and the  Docker image and testing from the source code
* short tutorial on [Gradle](Gradle)

### Graph Store Connection

The GM API requires a connection with the [Graph Store](Graph-Store.md), such a connection is specified either in the indexing or clustering or linkage requests.
A graph store connection is required:
* obtained provenance graph information;
* clustering IDs based on input Working Graphs;
* using the data for the indexing, in order to place data in the distribution component.

### Elasticsearch Connection

In order to process data in Elasticsearch 5.1 as JSON-LD there are a few prerequisites:
* plugin field should be: `python`;
* `query` field needs to contain a SPARQL query;
* Data needs to be in JSON-LD compact format;
* Data needs to be have a `@context` included;
* Index and Topics need to be specified;
* Indexing documents based on specific resource type: e.g. 1 publication indexed at a time.

In order to process data in Elasticsearch 1.3.4 and Elasticsearch Siren plugin:
* plugin field should be: `java`;
* `query` field needs to contain a string version of the JSON indexing;
* endpoint for the Elasticsearch should be 9300 for the bulk import;
* in the docker image there shoul be the `gc-rdf2json-indexer.jar`.

### UVProvovenance API Connection

In order to process the provenance information and update the Graph Store with it, the Graph Manager API pulls the data from the [UVProvovenance API](UVProvovenance-API.md) upon calling the `/prov?start=true` endpoint.

## Consuming the GM-API

The Graph Manager API has the following endpoints:
* `index` - on given indexing parameters index the data from the Graph Store and insert it into Distribution Component endpoint (POST method available);
* `index/{id}` - retrieve the status of a previously processed index - with the ID - or delete that index data (GET and DELETE methods available);
* `cluster` - cluster the IDs from the working data in the Graph Store (POST method available);
* `link` - start a linking job with a specific strategy (POST method available);
* `link/{id}` - retrieve the liking strategy with an ID or delete it (GET and DELETE methods available);
* `linkstrategy` - retrieve all the URIs of linking strategies from the Graph Store (GET method available);
* `linkstrategy/{id}` - retrieve all the URIs of linking strategies from the Graph Store (GET method available);
* `prov` - retrieve provenance from the UVProvovenance API and update the Graph Store (GET method available).
* `health` - responds with `200` so that we know the service is alive.

When the response from an endpoint returns a JSON with a key `status`, there can be some times values: `Done`, `WIP` (Work In Progress), `Error` (some times a specific error).

The functionality provided by the endpoints is intertwined as illustrated below, and while the endpoints can be consumed separately the vision is to have the functionality for `cluster`, `prov` and `link` scheduled and better managed via the workflow-component.
```
               +--------------+
               |              |
               | linkstrategy |
               |              |
               +--------+-----+
                        |
+---------+             |
|         +-----------+ |
| cluster |           | |
|         |     +-----v-v--+          +---------+
+---------+     |          |          |         |
                |   link   +---------->  index  |
+---------+     |          |          |         |
|         |     +----------+          +-----^---+
|  prov   |                                 |
|         +---------------------------------+
+---------+
```

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4302/health reponds with `200` in order for other applications to know the service is running.

### index Endpoint

More details on how the information is processed and indexed are available at: [How the Information Processed and Indexed](#how-the-information-processed-and-indexed).

Different calls specific to index endpoint:
* `index` POST API:
  * simple: http://localhost:4302/0.1/index
  * POST request is validated against a schema and it has the following structure (in the indexing key everything except the `filter` key is optional"):
  ```json
  {
    "plugin": "python",
    "targetEndpoint": {
        "host":"localhost",
        "port": 9200
    },
    "indexing": {
        "index": "default",
        "resourceType": "work",
        "indexingID": "dcterms:identifier",
        "filter": "{\n  \"@context\": {\n    \"@vocab\": \"http://data.hulib.helsinki.fi/attx/\",\n    \"dcterms\": \"http://purl.org/dc/terms/\",\n    \"rdf\": \"http://www.w3.org/1999/02/22-rdf-syntax-ns#\",\n    \"rdfs\": \"http://www.w3.org/2000/01/rdf-schema#\",\n    \"xsd\": \"http://www.w3.org/2001/XMLSchema#\",\n    \"attx-types\": \"http://data.hulib.helsinki.fi/attx/types/\",\n    \"attx-types:Infrastructure\": {\n    \"@type\": \"@id\"\n  },\n    \"attx:hasclusteredID\": {\n      \"@id\": \"http://data.hulib.helsinki.fi/attx/id\"\n    }\n    ,\n    \"dcterms:identifier\": {\n      \"@type\": \"@id\",\n      \"@id\": \"http://purl.org/dc/terms/identifier\"\n    }\n  },\n  \"@type\": \"attx-types:Infrastructure\"\n}"
    },
    "graphStore": {
        "endpoint": {
            "host":"localhost",
            "port":3030,
            "dataset": "ds"
        },
        "graphs":["http://data.hulib.helsinki.fi/attx/ids", "http://data.hulib.helsinki.fi/attx/work1"]
    }
  }
  ```
  Another variant to obtain the indexing is via the JAVA indexing processor. The body of the request should be unsing the following structure.

  ```json
  {
    "plugin": "java",
    "targetEndpoint": {
        "host": "localhost",
        "port": 9200
    },
    "indexing": {
        "filter": "{\"configs\":[{\"index\":\"uc1\",\"targets\":{\"http://data.hulib.helsinki.fi/attx/types/Infrastructure\":{\"type\":\"infra\",\"indexDoc\":{\"dataProperties\":[{\"sourceProperty\":\"http://purl.org/dc/terms/title\",\"targetProperty\":\"title\",\"isFunctional\":\"true\"}],\"multiLingualDataProperties\":[],\"objectProperties\":[{\"sourceProperty\":\"http://purl.org/dc/terms/identifier\",\"targetProperty\":\"identifier\",\"isFunctional\":\"true\"}]}},\"http://data.hulib.helsinki.fi/attx/types/Publication\":{\"type\":\"pub\",\"indexDoc\":{\"dataProperties\":[{\"sourceProperty\":\"http://purl.org/dc/terms/title\",\"targetProperty\":\"title\",\"isFunctional\":\"true\"},{\"sourceProperty\":\"http://purl.org/dc/terms/identifier\",\"targetProperty\":\"identifier\",\"isFunctional\":\"true\"}],\"multiLingualDataProperties\":[],\"objectProperties\":[]}}}}]}"
    },
    "graphStore": {
        "endpoint": {
            "host": "localhost",
            "port": 3030,
            "dataset": "ds"
        },
        "graphs": ["http://data.hulib.helsinki.fi/attx/ids", "http://data.hulib.helsinki.fi/attx/work1"]
    }
  }

  ```
  * Responses: `202` or `400`. When the request is successful the following response is issued:
  ```json
  {
    "id": 123,
    "status": "WIP"
  }
  ```
  When the response status code is `400` or a bad request (bad request due to validation issues), a response under the following format is issued:
  ```json
  {
    "title": "Invalid data",
    "description": "Could not properly parse the provided data as JSON"
  }
  ```
  or (a message where the description depends on the error message):
  ```json
  {
    "title": "Failed data validation",
    "description": "THE DESCRIPTION CAN VARY"
  }
  ```
* `index/{id}` GET API:
  * simple:  http://localhost:4302/0.1/index/{id}
  * Responses: `200` or `410` (for the `410` the body of the response is empty). The following response the request is completed successfully (a.k.a. `200`):
  ```json
  {
    "id": 123,
    "status": "Done"
  }
  ```
* `index/{id}` DELETE API:
  * simple:  http://localhost:4302/0.1/index/{id}
  * Responses: `200`
  ```json
  {
    "deletedID": 123
  }
  ```

### cluster Endpoint

The cluster endpoint:
* `cluster` POST API:
  * simple: http://localhost:4302/0.1/cluster with the following body (if the body format is badly formatted the response will be `400`):
  ```json
  {
    "graphStore": {
        "host": "localhost",
        "port": 3030,
        "dataset": "ds"
    }
  }
  ```
  * Responses: `200`. In case of success the response should be a message containing:
  ```json
  {
    "IDCount": 123,
    "status": "Processed"
  }
  ```
  In case the Clustering of IDs is not possible the response status code is `422` and the message is (title always the same the description can differ):
  ```json
  {
    "title": "Unprocessable ID Clustering",
    "description": "Could not process the clustering of ids."
  }
  ```

### link Endpoint

* `link` POST API:
  * simple: http://localhost:4302/0.1/link with the following body:
  ```json
  {
    "strategy": {
        "uri": "http://data.hulib.helsinki.fi/attx/idstrategy1",
        "output": "http://data.hulib.helsinki.fi/attx/work3"
    },
    "graphStore": {
        "endpoint": {
            "host":"localhost",
            "port":3030,
            "dataset": "ds"
        },
        "graphs":["http://data.hulib.helsinki.fi/attx/work2", "http://data.hulib.helsinki.fi/attx/work1"]
    }
  }
  ```
  **From the request body above the keys `output` and `graphs` can be omitted. For the `output` the implementation will generate a random 16 digit id. Meanwhile omitting the `graphs` will trigger the gm_API to use the whole ATTX clustered IDs graph instead of the clustered IDs graph generated from a specific list of graphs.**

  The `link` endpoint will also generated associated provenance for the link activity and update the provenance graph in the Graph Store.

  * Responses: `202` Accepted or `400` not allowed with the body:
  ```json
  {
    "id": 123,
    "status": "WIP"
  }
  ```

The strategy for linking graphs is detailed at: [Linking Graphs](Linking-graphs.md)

* `link/{id}` GET API:
  * simple:  http://localhost:4302/0.1/link/{id}
  * Responses: `200` or `410` (for the `410` the body of the response is empty). The following response the request is completed successfully (a.k.a. `200`):
  ```json
  {
    "id": 123,
    "status": "Done"
  }
  ```
* `link/{id}` DELETE API:
  * simple:  http://localhost:4302/0.1/link/{id}
  * Responses: `200`
  ```json
  {
    "deletedID": 123
  }
  ```

### linkstrategy Endpoint

* `linkstrategy`GET API:
  * simple:  http://localhost:4302/0.1/linkstrategy the response will be `200` upon successful retrieving the strategies or `404` if no strategies exist and the body will contain an array of strategy URIs and titles:

  ```json
  [
      {
        "title": "IDs based Linking Strategy",
        "uri": "http://data.hulib.helsinki.fi/attx/idstrategy1"
      },
      {
        "title": "IDs v2 based Linking Strategy",
        "uri": "http://data.hulib.helsinki.fi/attx/idstrategy2"
      }
  ]
  ```

   * to configure the endpoints for the Graph Store the following parameters can be used:
     * `?graphStore=http://fusekidockerimagename:port/dataset` - the value can be a docker image name alongside the port and dataset (e.g. `ds` for production and `test` for testing) - if not specified it defaults to `http://localhost:3030/ds`.
* `linkstrategy/{id}` GET API:
  * simple:  http://localhost:4302/0.1/linkstrategy/{id} where `{id}` is the ID of the strategy not the full URI (e.g. `idstrategy1` in `http://data.hulib.helsinki.fi/attx/idstrategy1`) the response will be `200` upon successfully retrieving the strategy or `404` if that strategies does not exist and the body will contain the URIs and the parameters of the strategy:

  ```json
    {
     "parameters": {
        "description": "Linking Strategy based on IDs clustering and specifying required datasets.",
        "hasParameters": "List of parameters",
        "hasStrategyType": "SPARQL",
        "query": "prefix skos: <http://www.w3.org/2004/02/skos/core#>\n prefix attx: <http://data.hulib.helsinki.fi/attx/>\n construct { ?r1 skos:exactMatch ?r2} where { ?r1 attx:id ?id .\n  ?r2 attx:id ?id .\n  filter(?r1 != ?r2)\n}",
        "title": "IDs based Linking Strategy"
     },
     "uri": "http://data.hulib.helsinki.fi/attx/idstrategy1"
    }
  ```

   * to configure the endpoints for the Graph Store the following parameters can be used:
     * `?graphStore=http://fusekidockerimagename:port/dataset` - the value can be a docker image name alongside the port and dataset (e.g. `ds` for production and `test` for testing) - if not specified it defaults to `http://localhost:3030/ds`.

### prov Endpoint

The prov endpoint works as described below:
* `prov` GET API:
  * simple: http://localhost:4302/0.1/prov - retrieve status and see the last run
  * start provenance: http://localhost:4302/0.1/prov?start=true - to do a provenance update (there are two values to start true or false, default is false)
  * start provenance from specific date: http://localhost:4302/0.1/prov?start=true&modifiedSince=2017-02-09T13:05:50Z - to do a provenance update (using the format: `%Y-%m-%dT%H:%M:%SZ`)
  * to configure the endpoints for the UVProvenance-API and Graph store the following parameters can be used:
    * `?wfapi=http://wfdockerimagename:port/apiversion` - the value can be a docker image name alongside the port and version - if not specified it defaults to `http://localhost:4302/0.1`;
    * `?graphStore=http://fusekidockerimagename:port/dataset` - the value can be a docker image name alongside the port and dataset (e.g. `ds` for production and `test` for testing) - if not specified it defaults to `http://localhost:3030/ds`.
    * whole request: http://localhost:4302/0.1/prov?start=true&?wfapi=http://wfdockerimagename:port/apiversion&?graphStore=http://fusekidockerimagename:port/dataset
  * Responses: `200` or `404`. In the case of success the response should be a message containing:

  ```json
  {
    "lastStart": "2017-02-15T08:14:14Z",
    "status": "Done"
  }
  ```
The `lastStart` parameter has the format `%Y-%m-%dT%H:%M:%SZ`.
In case no provenance update has been performed it will return `404` Not Found. In case there is nothing to update it will return the last provenance update run.
Another response status code is `502` Bad Gateway in case the Graph Manager cannot connect to any of the required endpoints. And the response has the following body:

```json
    {
      "title": "Failed to do Graph Store update."
    }
```

or

```json
    {
      "title": "Failed to do the get provenance from UVProvenance-API update."
    }
```

## Processing and Indexing Graph Data

The information is processed and indexed according to the plugins specification. Plugins provided with the current version of the implementation are:
* `python` - simple JSON indexing of a resource/document (a particular set of triples associated with a subject in a graph) which was processed by a JSON-LD frame;
* `Java` - makes use of ElasticSearch Siren plugin to index and structure data.

Proposed plugins:
* `python-ld` - framed JSON-LD will be indexed in Elasticsearch and each indexed resource/document would take shape of sub-graph (or tree based on the subject/JSON-LD frame).

### Elasticsearch 5.x and JSON derived from JSON-LD frame

Given a JSON-LD frame (included in the POST `index` request):

```json
{
  "@context": {
    "@vocab": "http://data.hulib.helsinki.fi/attx/",
    "dcterms": "http://purl.org/dc/terms/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "attx-types": "http://data.hulib.helsinki.fi/attx/types/",
    "attx-types:Infrastructure": {
    "@type": "@id"
  },
    "attx:hasclusteredID": {
      "@id": "http://data.hulib.helsinki.fi/attx/id"
    }
    ,
    "dcterms:identifier": {
      "@type": "@id",
      "@id": "http://purl.org/dc/terms/identifier"
    }
  },
  "@type": "attx-types:Infrastructure"
}
```
The resulting JSON-LD would have the following structure:

```json
{
  "@context": {
    "@vocab": "http://data.hulib.helsinki.fi/attx/",
    "dcterms": "http://purl.org/dc/terms/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "attx-types": "http://data.hulib.helsinki.fi/attx/types/",
    "attx-types:Infrastructure": {
      "@type": "@id"
    },
    "attx:hasclusteredID": {
      "@id": "http://data.hulib.helsinki.fi/attx/id"
    },
    "dcterms:identifier": {
      "@type": "@id",
      "@id": "http://purl.org/dc/terms/identifier"
    }
  },
  "@graph": [
    {
      "@id": "http://data.hulib.helsinki.fi/attx/3/work/infra/urn:nbn:fi:research-infras-201607251",
      "@type": "attx-types:Infrastructure",
      "attx:hasclusteredID": {
        "@id": "urn:nbn:fi:research-infras-201607251"
      },
      "dcterms:identifier": "urn:nbn:fi:research-infras-201607251",
      "dcterms:title": "European Social Survey"
    },
    {
      "@id": "http://data.hulib.helsinki.fi/attx/3/work/infra/urn:nbn:fi:research-infras-2016072510",
      "@type": "attx-types:Infrastructure",
      "attx:hasclusteredID": {
        "@id": "urn:nbn:fi:research-infras-2016072510"
      },
      "dcterms:identifier": "urn:nbn:fi:research-infras-2016072510",
      "dcterms:title": "Finnish Marine Research Infrastructure"
    }]
  }
```

### Elasticsearch 5.x indexing JSON-LD

TBD

### Elasticsearch 1.3.4 and Siren plugin

Details available at: [ElasticSearch with Siren](ElasticSearch-with-Siren.md)

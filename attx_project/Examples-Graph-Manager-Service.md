# Graph Manager Service Examples

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Retrieve Graph Dataset](#retrieve-graph-dataset)
  - [Query Graph Dataset](#query-graph-dataset)
  - [Update Graph Dataset](#update-graph-dataset)
  - [Replace Graph Dataset](#replace-graph-dataset)
    - [Replace using Turtle content](#replace-using-turtle-content)
    - [replace using files paths](#replace-using-files-paths)

<!-- TOC END -->


## Retrieve Graph Dataset

## Query Graph Dataset

## Update Graph Dataset

## Replace Graph Dataset

Example of replace name graph datasets are below. The replace named graph dataset implements a SPARQL 1.1 Graph Store HTTP Protocol [HTTP PUT Method](https://www.w3.org/TR/sparql11-http-rdf-update/#http-put)

### Replace using Turtle content

```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "replaceds"
        }
    },
    "payload": {
        "graphManagerInput": {
            "contentType": "text/turtle",
            "inputType": "Data",
            "namedGraph": "default",
            "input": "<http://example/egbook3> <http://purl.org/dc/elements/1.1/title> \"This is an example title 3153196587138\".",
            "activity": "replace"
        }
    }
}
```
Result:
```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "replaceds"
        }
    },
    "payload": {
        "graphManagerOutput": {
            "count": 1,
            "quadCount": 0,
            "tripleCount": 1
        }
    }
}
```
### replace using files paths

```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "replaceds"
        }
    },
    "payload": {
        "graphManagerInput": {
            "contentType": "text/turtle",
            "inputType": "URI",
            "namedGraph": "default",
            "input": "file:///home/local/example-data/example.ttl",
            "activity": "replace"
        }
    }
}
```

Result:
```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "replaceds"
        },
        "agent": {
            "ID": "GMAPI",
            "role": "storage"
        },
        "activity": {
            "title": "Replace data in the graph",
            "type": "ServiceExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00",
            "status": "SUCCESS"
        },
        "input": [
          {
            "key": "inputGraphs",
            "role": "tempDataset"
          }
      ],
      "output": [
        {
          "key": "outpuGraphs",
          "role": "Dataset"
        }
      ]             
    },
    "payload": {
        "inputGraphs": "file:///home/local/example-data/example.ttl",
        "outpuGraphs": "default"
    }
}

```

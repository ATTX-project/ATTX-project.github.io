# Graph Manager Service Examples

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Retrieve Graph Dataset](#retrieve-graph-dataset)
  - [Query Graph Dataset](#query-graph-dataset)
  - [Add Graph Dataset](#add-graph-dataset)
  - [Replace Graph Dataset](#replace-graph-dataset)
    - [Replace using Turtle content](#replace-using-turtle-content)    
  - [Update Graph Dataset](#update-graph-dataset)
  - [Version Graph Dataset](#version-graph-dataset)

<!-- TOC END -->


## Retrieve Graph Dataset

Retrieve returns the content of the source graphs in a given format. Output can be store to a temporary file or returned as message payload.

Possible values for outputType:
* text/turtle


### Retrive with URI output

```json

{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "stepid"
        }
    },
    "payload": {
        "graphManagerInput": {
          "activity": "retrieve",
          "contentType": "text/turtle",
          "outputType": "URI",           
          "sourceGraphs":
          [
              "http://work/dataset1",
              "http://work/dataset2"
          ]
        }
    }    
}
```
Result:
```json
{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "stepid"
        }
    },
    "payload": {
        "graphManagerOutput": {
          "contentType": "text/turtle",
          "outputType": "URI",            
          "output": "file://temp/file1"
        }
    }
}
```

### Retrive with Data output

```json

{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "stepid"
        }
    },
    "payload": {
        "graphManagerInput": {
          "activity": "retrieve",
          "contentType": "application/n-triples",
          "outputType": "Data",           
          "sourceGraphs":
          [
              "http://work/dataset3"              
          ]
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
          "contentType": "application/n-triples",
          "outputType": "Data",            
          "output": "<http://example.org/#spiderman> <http://www.perceive.net/schemas/relationship/enemyOf> <http://example.org/#green-goblin> ."
        }
    }
}
```
## Query Graph Dataset

Could this work with multiple source graphs?

```json
{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "stepid"
        }
    },
    "payload": {
        "graphManagerInput": {
          "activity": "query",
          "outputType": "URI",           
          "sourceGraph": "http://work/dataset3",
          "input": "SELECT ?subject ?predicate ?object WHERE { ?subject ?predicate ?object} LIMIT 25"            
        }
    }
}

```

## Add Graph Dataset

Adds the source triples to the target graph and does not modify the existing data.

```json

{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "stepid"
        }
    },
    "payload": {
        "graphManagerInput": {
          "activity": "add",
          "targetGraph": "http://work/dataset1",
          "sourceData":
          [
            {
              "contentType": "text/turtle",
              "inputType": "URI",            
              "input": "file://temp/file1"
            },
            {
              "contentType": "application/n-triples",
              "inputType": "Data",            
              "input": "<http://example.org/#spiderman> <http://www.perceive.net/schemas/relationship/enemyOf> <http://example.org/#green-goblin> ."

            }            
          ]
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

## Replace Graph Dataset

Replace graph takes the input triples and adds them to the target graph. All possible previous content in the graph will be deleted. If there are more that one sources, the resulting graph contain distinct triples from all of them.

Example of replace name graph dataset are below.

### Replace using content from two sources

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
          "activity": "replace",
          "targetGraph": "http://work/dataset1",
          "sourceData":
          [
            {
              "contentType": "text/turtle",
              "inputType": "URI",            
              "input": "file://temp/file1"
            },
            {
              "contentType": "application/n-triples",
              "inputType": "Data",            
              "input": "<http://example.org/#spiderman> <http://www.perceive.net/schemas/relationship/enemyOf> <http://example.org/#green-goblin> ."

            }            
          ]
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

## Update Graph Dataset

**TODO**

## Version Graph Dataset

**TODO**

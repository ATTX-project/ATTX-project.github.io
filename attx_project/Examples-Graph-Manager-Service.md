# Graph Manager Service examples

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Retrieve Graph Dataset](#retrieve-graph-dataset)
    - [Retrive with URI output](#retrive-with-uri-output)
    - [Retrive with Data output](#retrive-with-data-output)
  - [Query Graph Dataset](#query-graph-dataset)
  - [Construct Graph Dataset](#construct-graph-dataset)
  - [Add Graph Dataset](#add-graph-dataset)
  - [Replace Graph Dataset](#replace-graph-dataset)
    - [Replace using content from two sources](#replace-using-content-from-two-sources)
  - [Patch Graph Dataset](#patch-graph-dataset)
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
          "contentType": "application/sparql-results+xml",
          "outputType": "Data",           
          "sourceGraphs": ["http://work/dataset2", "default"],
          "input": "SELECT ?subject ?predicate ?object WHERE { ?subject ?predicate ?object} LIMIT 25"            
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
          "contentType": "application/sparql-results+xml",
          "outputType": "Data",            
          "output": ""
        }
    }
}
```

## Construct Graph Dataset

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
      "activity": "construct",
      "contentType": "text/turtle",
      "outputType": "Data",
      "sourceGraphs": ["http://test.data", "default"],
      "input": "PREFIX smth:<http://www.snee.com/ns/demo#> CONSTRUCT { ?p smth:hasGrandfather ?g . } WHERE {?p      smth:hasParent ?parent . ?parent smth:hasParent ?g . ?g      smth:gender    smth:male .}"
    }
}
}
```
Result:
```json
{
    "payload": {
        "graphManagerOutput": {
            "contentType": "text/turtle",
            "output": "@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .\n@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .\n@prefix smth: <http://www.snee.com/ns/demo#> .\n@prefix xml: <http://www.w3.org/XML/1998/namespace> .\n@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .\n\nsmth:Jane smth:hasGrandfather smth:Pat .\n\nsmth:Mike smth:hasGrandfather smth:Pat .\n\n",
            "outputType": "Data"
        }
    },
    "provenance": {
        "agent": {
            "ID": "GraphManager",
            "role": "storage"
        },
        "context": {
            "activityID": "1",
            "stepID": "stepid",
            "workflowID": "wf"
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
        "graphManagerOutput": "success"
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
        "graphManagerOutput": "success"
    }
}
```

## Patch Graph Dataset

**TODO**

## Version Graph Dataset

**TODO**

# Ontology Service examples


## Message Broker Communication

Request

```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "validation"
        }
    },
    "payload": {
        "ontologyServiceInput": {
            "activity": "infer",
            "sourceData": {
                "schemaGraph": "file:///resources/owlDemoSchema.ttl",
                "dataGraph":"file:///resources/owlDemoData.ttl"

            }
        }
    }
}
```

Response

```json
{
    "provenance": {
        "context": {
            "activityID": "1",
            "stepID": "replaceds",
            "workflowID": "ingestionwf"
        },
        "agent": {
            "role": "index",
            "ID": "IndexingService"
        }
    },
    "payload": {
        "status": "success",
        "statusMessage": "",
        "ontologyServiceOutput": "file:///attx-sb-shared/ontologyservice/file/results.ttl"
    }
}
```

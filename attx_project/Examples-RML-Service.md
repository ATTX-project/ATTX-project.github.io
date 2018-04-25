# RML Service examples

## Message Broker Communication

### Transform using inline input

Request

```json
{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "step"
        }
    },
    "payload": {
        "type": "URI",
        "input": "file:///attx-sb-shared/temp/file.json",
        "mapping": "mapping content in turtle"
    }    
}
```

Response

```json
{
    "provenance": {
        "context": {
            "workflowID": "wf",
            "activityID": 1,
            "stepID": "step"
        }
    },
    "payload": {
      "contentType": "text/turtle",
      "status": "SUCCESS",
      "statusMessage": "",
      "transformedDatasetURL": "file:///attx-sb-shared/temp/file_transformer.ttl"
    }    
}

```

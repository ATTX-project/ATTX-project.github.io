# Indexing Service examples

Request

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
        "indexingServiceInput": {
            "task": "replace",
            "targetAlias": ["test"],
            "sourceData": [{
                    "input": "file:///attx-sb-shared/temp/file.json",
                    "docType": "workflowingestionwf_activity1",
                    "useBulk": true,
                    "inputType": "URI"
                },
                {
                    "input": "file:///attx-sb-shared/temp/file1.json",
                    "docType": "workflow1_activity8",
                    "useBulk": true,
                    "inputType": "URI"
                }
            ]
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
        "status": "success"
    }
}
```

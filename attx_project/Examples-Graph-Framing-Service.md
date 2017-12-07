# Graph Framing Service examples

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
        "framingServiceInput": {
            "docType": "test",
            "ldFrame": "{\r\n  \"@context\": {\r\n    \"dc\": \"http:\/\/purl.org\/dc\/elements\/1.1\/\",\r\n    \"ex\": \"http:\/\/example.org\/vocab#\"\r\n  },\r\n  \"@type\": \"ex:Library\",\r\n  \"ex:contains\": {\r\n    \"@type\": \"ex:Book\",\r\n    \"ex:contains\": {\r\n      \"@type\": \"ex:Chapter\"\r\n    }\r\n  }\r\n}",
            "sourceData": [{
                "inputType": "URI",
                "contentType": "text/turtle",
                "input": "file:///attx-sb-shared/graph-data.ttl"
            },
            {
               "input":"http://data.hulib.helsinki.fi/prov_workflow1_activity6",
               "inputType":"Graph"
            }]
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
            "role": "LDframe",
            "ID": "GraphFraming"
        }
    },
    "payload": {
        "status": "success",
        "framingServiceOutput": {
            "output": "file:///attx-sb-shared/graphframing/db826a29bf5e4b208795cc00a72cc55d/4fa92c9cdb4211e7af16705a0f3940e5.json",
            "contentType": "elasticbulkfile",
            "outputType": "URI"
        }
    }
}
```

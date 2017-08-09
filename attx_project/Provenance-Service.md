<h1 style="color:red">DRAFT - work in progress</h1>
# Provenance Service

Example Provenance message:

```json
{
  "provenance" : {
    "context": {
      "workflowID": 1,
      "activityID": 1,
      "stepID": 1
    },
    "agent": {
      "ID": "agentID",
      "role": "agentRole"      
    },
    "activity": {
      "title": "activity title",      
      "type": "step",
      "description": "description",
      "status": "SUCCESS",
      "startTime": "2017-08-02T13:52:29+02:00",
      "endTime": "2017-08-02T13:52:29+02:00",
      "communication": [
        {          
          "role": "transformer",
          "input": {
            "frame": {
              "role": "Configuration"
            },
            "inputGraphs": {
              "role": "inputGraphs"
            }
          }
        }
      ]
    },
    "input": {
      "inputGraphs": {
        "role": "inputGraphs"
      }    
    },
    "output": {
      "outputGraphs": {
        "role": "outputGraphs"
      }
    }
  },  
  "payload": {
    "inputgraphs": "attx:dataset1",    
    "outputGraphs": "http://platform/public/rest/documents"

  }
}
```

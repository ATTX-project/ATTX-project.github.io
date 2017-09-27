## Generating Provenance Information

The sequence of messages below are meant to illustrate a series of services that communicate between each other and the provenance related messages are captured and converted to RDF (and illustrated for each message in Turtle format).

**Step 1a** workflow-started-prov
```json
{
    "provenance": {
        "context": {
          "workflowID": "ingestionwf",
          "activityID": "1"

        },
        "agent": {
          "ID": "UV",
          "role": "ETL"
        },
        "activity": {
            "title": "Ingestion workflow",
            "type": "WorkflowExecution",
            "startTime": "2017-08-02T13:52:29+02:00"
        },
        "input": [
          {
            "key": "inputDataset",
            "role": "Dataset"
          }
        ],
        "output": [
          {
            "key": "outputDataset",
            "role": "Dataset"
          }
        ]
    },
    "payload": {
        "inputDataset": "http://dataset/1",
        "outputDataset": "http://dataset/2"
    }
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1_UV a attxonto:WorkflowExecution,
        prov:Activity ;
    dcterms:title "Ingestion workflow" ;
    prov:generated attx:workflowingestionwf_activity1_UV_outputDataset ;
    prov:qualifiedAssociation attx:association_8f964bd174e90db0b452e67f98948c42 ;
    prov:qualifiedGeneration attx:generated_7b9e078099d96962ae83f608c08d2e96 ;
    prov:qualifiedUsage attx:used_238d7310e5d052ba188efdf006e00705 ;
    prov:startedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:used attx:workflowingestionwf_activity1_UV_inputDataset ;
    prov:wasAssociatedWith attx:UV .

attx:ETL a prov:Role .

attx:association_8f964bd174e90db0b452e67f98948c42 a prov:Association ;
    prov:agent attx:UV ;
    prov:hadPlan attx:workflowingestionwf_activity1 ;
    prov:hadRole attx:ETL .

attx:generated_7b9e078099d96962ae83f608c08d2e96 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_UV_outputDataset ;
    prov:hadRole attx:Dataset .

attx:used_238d7310e5d052ba188efdf006e00705 a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_UV_inputDataset ;
    prov:hadRole attx:Dataset .

attx:Dataset a prov:Role .

attx:UV a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_UV_inputDataset a prov:Entity ;
    dcterms:source "http://dataset/1" .

attx:workflowingestionwf_activity1_UV_outputDataset a prov:Entity ;
    dcterms:source "http://dataset/2" .
```


**Step 1b** harvestData-prov
```json
{
    "provenance": {
        "context": {
          "workflowID": "ingestionwf",
          "activityID": "1",
          "stepID": "harvestData"
        },
        "agent": {
          "ID": "UV",
          "role": "ETL"      
        },  
        "activity": {
            "title": "Harvest data",
            "type": "StepExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00",
            "status": "SUCCESS"
        },
        "input": [
          {
            "key": "harvestConfiguration",
            "role": "StepConfiguration"
          }
        ],
        "output": [
          {
            "key": "harvestedContent",
            "role": "DatasetContent"
          }
        ]       
    },
    "payload": {
        "harvestConfiguration": {
            "apiURL": "http://data/api",
            "endpoint": "/stocks",
            "query": "*"            
        }        
    }
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1 a attxonto:Workflow,
        prov:Plan ;
    pwo:hasStep attx:workflowingestionwf_activity1_stepharvestData_UV .

attx:DatasetContent a prov:Role .

attx:ETL a prov:Role .

attx:StepConfiguration a prov:Role .

attx:association_9cf07b13957d30c1dff6f6a88126cb4b a prov:Association ;
    prov:agent attx:UV ;
    prov:hadRole attx:ETL .

attx:generated_fdebbcfd7bbb2b132896ca57fd37d838 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_stepharvestData_UV_harvestedContent ;
    prov:hadRole attx:DatasetContent .

attx:used_4b49efb352be2a9e004afef3d85fdda7 a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_stepharvestData_UV_harvestConfiguration ;
    prov:hadRole attx:StepConfiguration .

attx:workflowingestionwf_activity1_stepharvestData_UV a attxonto:StepExecution,
        prov:Activity ;
    dcterms:title "Harvest data" ;
    prov:endedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:generated attx:workflowingestionwf_activity1_stepharvestData_UV_harvestedContent ;
    prov:qualifiedAssociation attx:association_9cf07b13957d30c1dff6f6a88126cb4b ;
    prov:qualifiedGeneration attx:generated_fdebbcfd7bbb2b132896ca57fd37d838 ;
    prov:qualifiedUsage attx:used_4b49efb352be2a9e004afef3d85fdda7 ;
    prov:startedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:used attx:workflowingestionwf_activity1_stepharvestData_UV_harvestConfiguration ;
    prov:wasAssociatedWith attx:UV .

attx:UV a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_stepharvestData_UV_harvestConfiguration a prov:Entity ;
    dcterms:source "{u'query': u'*', u'endpoint': u'/stocks', u'apiURL': u'http://data/api'}" .

attx:workflowingestionwf_activity1_stepharvestData_UV_harvestedContent a prov:Entity .
```


**Step 2** transformToRDF-prov
```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "tranformToRDF"
        },
        "agent": {
            "ID": "UV",
            "role": "ETL"
        },
        "activity": {
            "title": "Transform to RDF",
            "type": "StepExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00",
            "status": "SUCCESS",
            "communication": [{
                "agent": "RMLService",
                "role": "transformer",
                "input": []
            }]
        },

        "input": [{
            "key": "harvestedContent",
            "role": ""
        }],
        "output": [{
            "key": "transformerData",
            "role": "tempDataset"
        }]
    },
    "payload": {}
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1 a attxonto:Workflow,
        prov:Plan ;
    pwo:hasStep attx:workflowingestionwf_activity1_steptranformToRDF_UV .

attx:ETL a prov:Role .

attx:RMLService a attxonto:Artifact,
        prov:Agent .

attx:association_9cf07b13957d30c1dff6f6a88126cb4b a prov:Association ;
    prov:agent attx:UV ;
    prov:hadRole attx:ETL .

attx:generated_8c7a94b79785a3e0a13b22d176194f34 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_steptranformToRDF_UV_transformerData ;
    prov:hadRole attx:tempDataset .

attx:tempDataset a prov:Role .

attx:transformer a prov:Role .

attx:workflowingestionwf_activity1_steptranformToRDF_RMLService a prov:Activity ;
    prov:wasAssociatedWith attx:RMLService .

attx:workflowingestionwf_activity1_steptranformToRDF_UV a attxonto:StepExecution,
        prov:Activity ;
    dcterms:title "Transform to RDF" ;
    prov:endedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:generated attx:workflowingestionwf_activity1_steptranformToRDF_UV_transformerData ;
    prov:qualifiedAssociation attx:association_9cf07b13957d30c1dff6f6a88126cb4b ;
    prov:qualifiedCommunication [ a prov:Communication ;
            prov:activity attx:workflowingestionwf_activity1_steptranformToRDF_RMLService ;
            prov:hadRole attx:transformer ] ;
    prov:qualifiedGeneration attx:generated_8c7a94b79785a3e0a13b22d176194f34 ;
    prov:startedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:used attx:workflowingestionwf_activity1_steptranformToRDF_UV_harvestedContent ;
    prov:wasAssociatedWith attx:UV .

attx:workflowingestionwf_activity1_steptranformToRDF_UV_harvestedContent a prov:Entity .

attx:UV a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_steptranformToRDF_UV_transformerData a prov:Entity .
```
**Step 2 - REPLY**
```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "stepID": "tranformToRDF"
        },
        "agent": {
            "ID": "RMLService",
            "role": "transformer"
        },
        "activity": {
            "title": "Transform Data",
            "type": "ServiceExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00",
            "status": "SUCCESS"
        },
        "input": [
          {
            "key": "inputGraphs",
            "role": "inputGraphs"
          }
        ],
        "output": [
          {
            "key": "transformerData",
            "role": "tempDataset"
          }
        ]                        
    },
    "payload": {
        "inputGraphs": "attx:dataset1",
        "transformerData": "attx:tempDataset"
    }
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1_steptranformToRDF_RMLService a attxonto:ServiceExecution,
        prov:Activity ;
    dcterms:title "Transform Data" ;
    prov:endedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:generated attx:workflowingestionwf_activity1_steptranformToRDF_RMLService_transformerData ;
    prov:qualifiedAssociation attx:association_01ed3446f49064dc2e2995af157ece48 ;
    prov:qualifiedGeneration attx:generated_8c7a94b79785a3e0a13b22d176194f34 ;
    prov:qualifiedUsage attx:used_ae325c51b1523371e8026f19b86e3e7e ;
    prov:startedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:used attx:workflowingestionwf_activity1_steptranformToRDF_RMLService_inputGraphs ;
    prov:wasAssociatedWith attx:RMLService .

attx:association_01ed3446f49064dc2e2995af157ece48 a prov:Association ;
    prov:agent attx:RMLService ;
    prov:hadRole attx:transformer .

attx:generated_8c7a94b79785a3e0a13b22d176194f34 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_steptranformToRDF_RMLService_transformerData ;
    prov:hadRole attx:tempDataset .

attx:inputGraphs a prov:Role .

attx:tempDataset a prov:Role .

attx:transformer a prov:Role .

attx:used_ae325c51b1523371e8026f19b86e3e7e a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_steptranformToRDF_RMLService_inputGraphs ;
    prov:hadRole attx:inputGraphs .

attx:RMLService a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_steptranformToRDF_RMLService_inputGraphs a prov:Entity ;
    dcterms:source "attx:dataset1" .

attx:workflowingestionwf_activity1_steptranformToRDF_RMLService_transformerData a prov:Entity ;
    dcterms:source "attx:tempDataset" .
```

**Step 3a** replaceDataset-prov
```json
{
    "provenance": {
        "context": {
          "workflowID": "ingestionwf",
          "activityID": 1,
          "stepID": "replaceds"
        },
        "agent": {
          "ID": "UV",
          "role": "ETL"
        },
        "activity": {
            "title": "Replace content of the existing dataset",
            "type": "StepExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00",
            "description": "something peachy",
            "status": "SUCCESS",
            "communication": [
                {
                    "agent": "GMAPI" ,
                    "role": "storage",
                        "input": [ {
                           "key": "transformerData",
                           "role": "tempDataset"
                         }]
        }]},
        "input": [
          {
            "key": "transformerData",
            "role": "tempDataset"
          }
        ],
        "output": [
          {
            "key": "outputDataset",
            "role": "Dataset"
          }
        ]
    },
   "payload": {
       "transformerData": "attx:tempDataset",
       "outputDataset": "http://dataset/2"
   }
}

```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1 a attxonto:Workflow,
        prov:Plan ;
    pwo:hasStep attx:workflowingestionwf_activity1_stepreplaceds_UV .

attx:Dataset a prov:Role .

attx:ETL a prov:Role .

attx:GMAPI a attxonto:Artifact,
        prov:Agent .

attx:association_9cf07b13957d30c1dff6f6a88126cb4b a prov:Association ;
    prov:agent attx:UV ;
    prov:hadRole attx:ETL .

attx:generated_7b9e078099d96962ae83f608c08d2e96 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_stepreplaceds_UV_outputDataset ;
    prov:hadRole attx:Dataset .

attx:storage a prov:Role .

attx:tempDataset a prov:Role .

attx:used_8c7a94b79785a3e0a13b22d176194f34 a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_stepreplaceds_UV_transformerData ;
    prov:hadRole attx:tempDataset .

attx:workflowingestionwf_activity1_stepreplaceds_GMAPI a prov:Activity ;
    prov:qualifiedUsage [ a prov:Usage ;
            prov:entity attx:workflowingestionwf_activity1_stepreplaceds_GMAPI_b39d544213c61593513e8771de3ade8c ;
            prov:hadRole attx:workflowingestionwf_activity1_tempDataset ] ;
    prov:used attx:workflowingestionwf_activity1_stepreplaceds_GMAPI_b39d544213c61593513e8771de3ade8c ;
    prov:wasAssociatedWith attx:GMAPI .

attx:workflowingestionwf_activity1_stepreplaceds_UV a attxonto:StepExecution,
        prov:Activity ;
    dcterms:description "something peachy" ;
    dcterms:title "Replace content of the existing dataset" ;
    prov:endedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:generated attx:workflowingestionwf_activity1_stepreplaceds_UV_outputDataset ;
    prov:qualifiedAssociation attx:association_9cf07b13957d30c1dff6f6a88126cb4b ;
    prov:qualifiedCommunication [ a prov:Communication ;
            prov:activity attx:workflowingestionwf_activity1_stepreplaceds_GMAPI ;
            prov:hadRole attx:storage ] ;
    prov:qualifiedGeneration attx:generated_7b9e078099d96962ae83f608c08d2e96 ;
    prov:qualifiedUsage attx:used_8c7a94b79785a3e0a13b22d176194f34 ;
    prov:startedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:used attx:workflowingestionwf_activity1_stepreplaceds_UV_transformerData ;
    prov:wasAssociatedWith attx:UV .

attx:workflowingestionwf_activity1_tempDataset a prov:Role .

attx:UV a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_stepreplaceds_UV_outputDataset a prov:Entity ;
    dcterms:source "http://dataset/2" .

attx:workflowingestionwf_activity1_stepreplaceds_UV_transformerData a prov:Entity ;
    dcterms:source "attx:tempDataset" .
```


**Step 3b**
```json
{
    "provenance": {
        "context": {
          "workflowID": "ingestionwf",
          "activityID": 1

        },
        "agent": {
          "ID": "UV",
          "role": "ETL"      
        },  
        "activity": {
            "title": "Ingestion workflow",
            "type": "WorkflowExecution",
            "endTime": "2017-08-02T13:52:29+02:00"
        },
        "input": [
          {
            "key": "inputDataset",
            "role": "Dataset"
          }
        ],
        "output": [
          {
            "key": "outputDataset",
            "role": "Dataset"
          }
        ]                                         
    },
    "payload": {
        "inputDataset": "http://dataset/1",
        "outputDataset": "http://dataset/2"
    }
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1_UV a attxonto:WorkflowExecution,
        prov:Activity ;
    dcterms:title "Ingestion workflow" ;
    prov:endedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:generated attx:workflowingestionwf_activity1_UV_outputDataset ;
    prov:qualifiedAssociation attx:association_8f964bd174e90db0b452e67f98948c42 ;
    prov:qualifiedGeneration attx:generated_7b9e078099d96962ae83f608c08d2e96 ;
    prov:qualifiedUsage attx:used_238d7310e5d052ba188efdf006e00705 ;
    prov:used attx:workflowingestionwf_activity1_UV_inputDataset ;
    prov:wasAssociatedWith attx:UV .

attx:ETL a prov:Role .

attx:association_8f964bd174e90db0b452e67f98948c42 a prov:Association ;
    prov:agent attx:UV ;
    prov:hadPlan attx:workflowingestionwf_activity1 ;
    prov:hadRole attx:ETL .

attx:generated_7b9e078099d96962ae83f608c08d2e96 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_UV_outputDataset ;
    prov:hadRole attx:Dataset .

attx:used_238d7310e5d052ba188efdf006e00705 a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_UV_inputDataset ;
    prov:hadRole attx:Dataset .

attx:Dataset a prov:Role .

attx:UV a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_UV_inputDataset a prov:Entity ;
    dcterms:source "http://dataset/1" .

attx:workflowingestionwf_activity1_UV_outputDataset a prov:Entity ;
    dcterms:source "http://dataset/2" .
```

**Step 3 - REPLY**
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
            "title": "Store data in the graph",
            "type": "ServiceExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00",
            "status": "SUCCESS"
        },
        "input": [
          {
            "key": "inputGraphs",
            "role": "inputGraphs"
          }
        ]        
    },
    "payload": {
        "inputGraphs": "attx:dataset1"
    }
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1_stepreplaceds_GMAPI a attxonto:ServiceExecution,
        prov:Activity ;
    dcterms:title "Store data in the graph" ;
    prov:endedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:qualifiedAssociation attx:association_1a7661cb146d8eabc4b156414840585a ;
    prov:qualifiedUsage attx:used_ae325c51b1523371e8026f19b86e3e7e ;
    prov:startedAtTime "2017-08-02T13:52:29+02:00"^^xsd:dateTime ;
    prov:used attx:workflowingestionwf_activity1_stepreplaceds_GMAPI_inputGraphs ;
    prov:wasAssociatedWith attx:GMAPI .

attx:association_1a7661cb146d8eabc4b156414840585a a prov:Association ;
    prov:agent attx:GMAPI ;
    prov:hadRole attx:storage .

attx:inputGraphs a prov:Role .

attx:storage a prov:Role .

attx:used_ae325c51b1523371e8026f19b86e3e7e a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_stepreplaceds_GMAPI_inputGraphs ;
    prov:hadRole attx:inputGraphs .

attx:GMAPI a attxonto:Artifact,
        prov:Agent .

attx:workflowingestionwf_activity1_stepreplaceds_GMAPI_inputGraphs a prov:Entity ;
    dcterms:source "attx:dataset1" .

```

**Step 4**
```json
{
    "provenance": {
        "context": {
            "workflowID": "ingestionwf",
            "activityID": 1,
            "step": "describeExternalDS"

        },
        "agent": {
            "ID": "UV",
            "role": "ETL"
        },
        "activity": {
            "title": "Ingestion workflow",
            "type": "DescribeStepExecution",
            "startTime": "2017-08-02T13:52:29+02:00",
            "endTime": "2017-08-02T13:52:29+02:00"
        },
        "input": [{
            "key": "inputDataset",
            "role": "Dataset"
        }],
        "output": [{
                "key": "outputDataset",
                "role": "Dataset"
            },
            {
                "key": "outputDataset2",
                "role": "Dataset"
            }
        ]
    },
    "payload": {
        "inputDataset": "attx://ds/2",
        "outputDataset": {
            "uri": "attx://ds/1",
            "title": "Harvested dataset",
            "description": "",
            "publisher": "UH",
            "license": "http://cc/0"
        },
        "outputDataset2": "attx://ds/3"
    }
}
```
Result in TURTLE format:
```turtle
attx:workflowingestionwf_activity1_UV prov:generated attx:workflowingestionwf_activity1_UV_outputDataset,
        attx:workflowingestionwf_activity1_UV_outputDataset2 ;
    prov:qualifiedGeneration attx:generated_155a0a0226f3b3adf45e7bfdc872f86b,
        attx:generated_7b9e078099d96962ae83f608c08d2e96 ;
    prov:qualifiedUsage attx:used_238d7310e5d052ba188efdf006e00705 ;
    prov:used attx:workflowingestionwf_activity1_UV_inputDataset .

attx:generated_155a0a0226f3b3adf45e7bfdc872f86b a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_UV_outputDataset2 ;
    prov:hadRole attx:Dataset .

attx:generated_7b9e078099d96962ae83f608c08d2e96 a prov:Generation ;
    prov:entity attx:workflowingestionwf_activity1_UV_outputDataset ;
    prov:hadRole attx:Dataset .

attx:used_238d7310e5d052ba188efdf006e00705 a prov:Usage ;
    prov:entity attx:workflowingestionwf_activity1_UV_inputDataset ;
    prov:hadRole attx:Dataset .

attx:workflowingestionwf_activity1_UV_inputDataset a attxonto:Dataset,
        prov:Entity ;
    dcterms:source "attx://ds/2" .

attx:workflowingestionwf_activity1_UV_outputDataset a attxonto:Dataset,
        prov:Entity ;
    attx:description "" ;
    attx:license "http://cc/0" ;
    attx:publisher "UH" ;
    attx:title "Harvested dataset" ;
    attx:uri "attx://ds/1" ;
    dcterms:source "{u'publisher': u'UH', u'description': u'', u'uri': u'attx://ds/1', u'license': u'http://cc/0', u'title': u'Harvested dataset'}" .

attx:workflowingestionwf_activity1_UV_outputDataset2 a attxonto:Dataset,
        prov:Entity ;
    dcterms:source "attx://ds/3" .

attx:Dataset a prov:Role .
```

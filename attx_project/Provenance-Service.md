<h1 style="color:red">DRAFT - work in progress</h1>
# Provenance Service

```json
{
    "id": {
        "activityID":134,
        "workflowID": 135,
        "agentID": "UVProv"
    },
    "agent": {
        "prov:hadRole": "API",
        "name": "UVProvenanceAPI",
        "dcterms:source": "unifiedviews",
        "prov:actedOnBehalfOf": "ProvenanceService"
    },
    "activity": {
        "executedSuccessful": true,
        "prov:startedAtTime": "2017-07-17T12:51:23+02:00",
        "prov:enededAtTime": "2017-07-17T12:54:51+02:00",
        "prov:used": "http://data.hulib.helsinki.fi/attx/wf1_dataset1",
        "prov:generated": "http://data.hulib.helsinki.fi/attx/wf1_dataset2"
    },
    "workflow": {
        "private": false,
        "dcterms:title": "Acquisition and Mapping the VIRTA pubs data.",
        "dcterms:description": "Example Workflow",
        "steps": [
            {
                "stepID": 12515,
                "isFirstStep": true,
                "dcterms:title": "VIRTA Publication Step",
                "dcterms:description": "Get the latest publication from VIRTA.",
            }
        ]
    }
}
```

```turtle
@prefix attxonto: <http://data.hulib.helsinki.fi/attx/onto#> .
@prefix attx: <http://data.hulib.helsinki.fi/attx/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix pwo: <http://purl.org/spar/pwo/> .

attx:worflow135_134_UVProv
  a attxonto:Workflow ;
  dcterms:title "Acquisition and Mapping the VIRTA pubs data." ;
  dcterms:description "Example Workflow"@en ;
  pwo:hasFirstStep attx:step135_12515 ;
  pwo:hasStep
    attx:step135_12515 .

attx:step135_12515
  a attxonto:Step ;
  dcterms:title "VIRTA Publication Step" ;
  dcterms:description "Get the latest publication from VIRTA."@en .

attx:activity135_134_UVProv
  a prov:Activity , attxonto:WorkflowExecution ;
  prov:startedAtTime  "2017-07-17T12:51:23+02:00"^^xsd:dateTime ;
  prov:endedAtTime "2017-07-17T12:54:51+02:00"^^xsd:dateTime ;
  prov:qualifiedAssociation [ a prov:Assocation ;
    prov:agent attx:UVProv ;
    prov:hadPlan attx:worflow135_134_UVProv ;] ;
  prov:used attx:dataset1 ;
  prov:generated attx:dataset2 .

attx:UVProv
  a prov:Agent ;
  prov:hadRole "API" ;
  prov:actedOnBehalfOf attx:ProvenanceService ;
  attxonto:usesArtifact attx:UVProvenanceAPI.

attx:UVProvenanceAPI
  a attxonto:Artifact ;
  dcterms:source "unifiedviews" .
```

```
activity(attx:activity135_134_UVProv,
2017-07-17T12:51:23, 2017-07-17T12:54:51)
wasAssociatedWith(attx:activity135_134_UVProv, attx:UVProv,
 attx:worflow135_134_UVProv, [prov:role='attx:API'])
used(attx:activity135_134_UVProv, attx:dataset1, -)
wasGeneratedBy(attx:dataset2, attx:activity135_134_UVProv, -)
```

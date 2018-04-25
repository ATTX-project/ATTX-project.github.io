# Examples of SHACL Validation Service

### Table of Contents
  - [Validation Service API](#validation-service-api)
  - [Validation Service Messages](#validation-service-messages)

## Validation Service API

```json
{"shapesGraph": "@prefix dash: <http:\/\/datashapes.org\/dash#> .\r\n@prefix rdf: <http:\/\/www.w3.org\/1999\/02\/22-rdf-syntax-ns#> .\r\n@prefix rdfs: <http:\/\/www.w3.org\/2000\/01\/rdf-schema#> .\r\n@prefix schema: <http:\/\/schema.org\/> .\r\n@prefix sh: <http:\/\/www.w3.org\/ns\/shacl#> .\r\n@prefix xsd: <http:\/\/www.w3.org\/2001\/XMLSchema#> .\r\n\r\nschema:PersonShape\r\n    a sh:NodeShape ;\r\n    sh:targetClass schema:Person ;\r\n    sh:property [\r\n        sh:path schema:givenName ;\r\n        sh:datatype xsd:string ;\r\n        sh:name \"given name\" ;\r\n    ] ;\r\n    sh:property [\r\n        sh:path schema:birthDate ;\r\n        sh:lessThan schema:deathDate ;\r\n        sh:maxCount 1 ;\r\n    ] ;\r\n    sh:property [\r\n        sh:path schema:gender ;\r\n        sh:in ( \"female\" \"male\" ) ;\r\n    ] ;\r\n    sh:property [\r\n        sh:path schema:address ;\r\n        sh:node schema:AddressShape ;\r\n    ] .\r\n\r\nschema:AddressShape\r\n    a sh:NodeShape ;\r\n    sh:closed true ;\r\n    sh:property [\r\n        sh:path schema:streetAddress ;\r\n        sh:datatype xsd:string ;\r\n    ] ;\r\n    sh:property [\r\n        sh:path schema:postalCode ;\r\n        sh:or ( [ sh:datatype xsd:string ] [ sh:datatype xsd:integer ] ) ;\r\n        sh:minInclusive 10000 ;\r\n        sh:maxInclusive 99999 ;\r\n    ] .",
"dataGraph": "@prefix ex: <http:\/\/example.org\/ns#> .\r\n@prefix rdf: <http:\/\/www.w3.org\/1999\/02\/22-rdf-syntax-ns#> .\r\n@prefix rdfs: <http:\/\/www.w3.org\/2000\/01\/rdf-schema#> .\r\n@prefix schema: <http:\/\/schema.org\/> .\r\n@prefix xsd: <http:\/\/www.w3.org\/2001\/XMLSchema#> .\r\n\r\nex:Bob\r\n    a schema:Person ;\r\n    schema:givenName \"Robert\" ;\r\n    schema:familyName \"Junior\" ;\r\n    schema:birthDate \"1971-07-07\"^^xsd:date ;\r\n    schema:deathDate \"1968-09-10\"^^xsd:date ;\r\n    schema:address ex:BobsAddress .\r\n\r\nex:BobsAddress\r\n    schema:streetAddress \"1600 Amphitheatre Pkway\" ;\r\n    schema:postalCode 9404 ."}
```

Result:

```turtle
@base          <http://example.org/random> .
@prefix schema: <http://schema.org/> .
@prefix ex:    <http://example.org/ns#> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .

[ a       <http://www.w3.org/ns/shacl#ValidationReport> ;
  <http://www.w3.org/ns/shacl#conforms>
          false ;
  <http://www.w3.org/ns/shacl#result>
          [ a       <http://www.w3.org/ns/shacl#ValidationResult> ;
            <http://www.w3.org/ns/shacl#focusNode>
                    ex:Bob ;
            <http://www.w3.org/ns/shacl#resultMessage>
                    "Value is not < value of schema:deathDate" ;
            <http://www.w3.org/ns/shacl#resultPath>
                    schema:birthDate ;
            <http://www.w3.org/ns/shacl#resultSeverity>
                    <http://www.w3.org/ns/shacl#Violation> ;
            <http://www.w3.org/ns/shacl#sourceConstraintComponent>
                    <http://www.w3.org/ns/shacl#LessThanConstraintComponent> ;
            <http://www.w3.org/ns/shacl#sourceShape>
                    []  ;
            <http://www.w3.org/ns/shacl#value>
                    "1971-07-07"^^xsd:date
          ] ;
  <http://www.w3.org/ns/shacl#result>
          [ a       <http://www.w3.org/ns/shacl#ValidationResult> ;
            <http://www.w3.org/ns/shacl#focusNode>
                    ex:Bob ;
            <http://www.w3.org/ns/shacl#resultMessage>
                    "Value does not have shape schema:AddressShape" ;
            <http://www.w3.org/ns/shacl#resultPath>
                    schema:address ;
            <http://www.w3.org/ns/shacl#resultSeverity>
                    <http://www.w3.org/ns/shacl#Violation> ;
            <http://www.w3.org/ns/shacl#sourceConstraintComponent>
                    <http://www.w3.org/ns/shacl#NodeConstraintComponent> ;
            <http://www.w3.org/ns/shacl#sourceShape>
                    []  ;
            <http://www.w3.org/ns/shacl#value>
                    ex:BobsAddress
          ]
] .

```

```json
{"dataGraph": "https://raw.githubusercontent.com/TopQuadrant/shacl/master/src/test/resources/sh/tests/core/property/class-001.test.ttl"}
```

Result:
```turtle
@base          <http://example.org/random> .
@prefix ex:    <http://datashapes.org/sh/tests/core/property/class-001.test#> .
@prefix sh:    <http://www.w3.org/ns/shacl#> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl:   <http://www.w3.org/2002/07/owl#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dash:  <http://datashapes.org/dash#> .

[ a            sh:ValidationReport ;
  sh:conforms  false ;
  sh:result    [ a                             sh:ValidationResult ;
                 sh:focusNode                  ex:InvalidResource1 ;
                 sh:resultMessage              "Value does not have class ex:SuperClass" ;
                 sh:resultPath                 ex:testProperty ;
                 sh:resultSeverity             sh:Violation ;
                 sh:sourceConstraintComponent  sh:ClassConstraintComponent ;
                 sh:sourceShape                ex:TestShape-testProperty ;
                 sh:value                      ex:InvalidResource1
               ] ;
  sh:result    [ a                             sh:ValidationResult ;
                 sh:focusNode                  ex:InvalidResource1 ;
                 sh:resultMessage              "Value does not have class ex:SuperClass" ;
                 sh:resultPath                 ex:testProperty ;
                 sh:resultSeverity             sh:Violation ;
                 sh:sourceConstraintComponent  sh:ClassConstraintComponent ;
                 sh:sourceShape                ex:TestShape-testProperty ;
                 sh:value                      "A string"
               ]
] .
```

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
        "validationServiceInput": {
            "sourceData": {
                "shapesGraph": "file:///attx-sb-shared/shapes-data.ttl",
                "dataGraph":"file:///attx-sb-shared/source-data.ttl"
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
        "validationServiceOutput": "file:///attx-sb-shared/validationservice/file/results.ttl"
    }
}

# Workflow API

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Overview](#overview)
  - [Building the WF-API](#building-the-wf-api)
    - [Database Connection](#database-connection)
  - [Consuming the WF-API](#consuming-the-wf-api)
    - [health Endpoint](#health-endpoint)
    - [workflow Endpoint](#workflow-endpoint)
    - [activity Endpoint](#activity-endpoint)
  - [How the Information is extracted](#how-the-information-is-extracted)

<!-- TOC END -->

## Overview

The workflow REST API has two endpoints along with a health check endpoint:
* `workflow` - retrieve all the workflows and associated steps from the ETL Artifact;
* `activity` - retrieve all successfully completed activities and associated datasets from the ETL Artifact;
* `health` - checks if the application is running.

The general purpose of the workflow API is to provide metadata for workflows and activities such that it exposes information that is according with the [Ontology/Data model](Data-Model.md). Such information could not easily be obtained directly from the [UnifiedViews REST API](https://grips.semantic-web.at/display/UDDOC/UnifiedViews+REST+API). Moreover the configuration necessary to extract this information is encoded in the Pipeline DPU configuration, as depicted in examples below.

Current version: `0.1` (URL for the endpoint should take into consideration for the API `http://host:4301/version/endpoint`)

## Building the WF-API

For building the WF-API see:
* [WF-API source](https://github.com/ATTX-project/workflow-component/blob/dev/wf-API/README.md) - building both the project and the  Docker image and testing from the source code
* short tutorial on [Gradle](Building-with-Gradle.md)

### Database Connection

In order to run the tests successfully but also the WF-API successfully one would require an existing MYSQL database connection. Such a connection can be configured in `database.conf` file.

At the same time it can be troubleshooted using the docker container:
* connect to the running WF-API Docker container `docker exec -it container-id /bin/sh` (replace `container-id` with the ID of the docker container)
* run python
* in the python shell:
  * `import pymysql as mysql`
  * `mysql.connect(host='addressOfDBServer', port=3306, user='username', passwd='password', db='database', charset='utf8')` - replace the variables in `''` with the correct information. Running this command with help with debugging mysql connection issues.

## Consuming the WF-API

As specified above the workflow API has two endpoints:
* `workflow` - retrieve all the workflows and associated steps from the ETL Artifact (only GET method possible);
* `activity` - retrieve all successfully completed activities and associated datasets from the ETL Artifact (only GET method possible).
* `health` - responds with `200` so that we know the service is alive.

### health Endpoint

* `health` GET API:
  * simple: http://localhost:4301/health responds with `200` in order for other applications to know the service is running.

### workflow Endpoint

* `workflow` GET API:
  * simple: http://localhost:4301/0.1/workflow
  * json-ld output: http://localhost:4301/0.1/workflow?format=json-ld
  * modifiedSince parameter URL encoded: http://localhost:4301/0.1/workflow?modifiedSince=2017-01-03T08%3A14%3A14Z the date-string should be in this format `%Y-%m-%dT%H:%M:%SZ` e.g. `2017-01-03T08:14:14Z`
  * all at once: http://localhost:4301/0.1/workflow?modifiedSince=2017-01-03T08%3A14%3A14Z&format=json-ld
  * Responses: if status codes are `304` No Modified or `405` Method Not Allowed the response body is empty. If the Response status is `200` it will return data specific to workflows:

  ```turtle
  @base <http://data.hulib.helsinki.fi/attx/> .
  @prefix attxonto: <http://data.hulib.helsinki.fi/attx/onto#> .
  @prefix attx: <http://data.hulib.helsinki.fi/attx/> .
  @prefix owl: <http://www.w3.org/2002/07/owl#> .
  @prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
  @prefix xml: <http://www.w3.org/XML/1998/namespace> .
  @prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
  @prefix prov: <http://www.w3.org/ns/prov#> .
  @prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
  @prefix dcterms: <http://purl.org/dc/terms/> .
  @prefix sd: <http://www.w3.org/ns/sparql-service-description#> .
  @prefix pwo: <http://purl.org/spar/pwo/> .

  attx:wf2
    a attxonto:Workflow ;
    dcterms:title "Acquisition and Mapping the VIRTA pubs data with CRIS data." ;
    dcterms:description "Workflow that uses VIRTA publication data in order to map titles and  identifiers from CRIS data for every publication and research fields."@en ;
    pwo:hasFirstStep attx:step1 ;
    pwo:hasStep
      attx:wf2_step1 ,
      attx:wf2_step2 ,
      attx:wf2_step3 ,
      attx:wf2_step4 ,
      attx:wf2_step5 .

  attx:wf2_step1
    a attxonto:Step ;
    dcterms:title "VIRTA Publication Input Step" ;
    dcterms:description "DPU that gets the latest publication from VIRTA using the file dump."@en ;
    pwo:hasNextStep attx:wf2_step2 .

  attx:wf2_step2
    a attxonto:Step ;
    dcterms:title "CRIS Acquisition Step" ;
    dcterms:description "DPU that acquires CRIS data."@en ;
    prov:used attx:work2 ;
    pwo:hasNextStep attx:wf2_step3 .

  attx:wf2_step3
    a attxonto:Step ;
    dcterms:title "Mapping Step" ;
    dcterms:description "DPU that maps the VIRTA data with CRIS data in order to match titles and identifies from the two data sets."@en ;
    pwo:hasNextStep attx:wf2_step4 .

  attx:wf2_step4
    a attxonto:Step ;
    dcterms:title "Adding wf2 Metadata Step" ;
    dcterms:description "DPU that add necessary Metadata, to the specify provenance information."@en ;
    pwo:hasNextStep attx:wf2_step5 .

  attx:wf2_step5
    a attxonto:Step ;
    dcterms:title "Combined Publication Output Step" ;
    dcterms:description "DPU that that contains the mapped data and writes it to the a file."@en .

  ```
### activity Endpoint

* `activity` GET API:
  * simple: http://localhost:4301/0.1/activity
  * json-ld output: http://localhost:4301/0.1/activity?format=json-ld
  * modifiedSince parameter URL encoded: http://localhost:4301/0.1/activity?modifiedSince=2017-01-03T08%3A14%3A14Z the date-string should be in this format `%Y-%m-%dT%H:%M:%SZ` e.g. `2017-01-03T08:14:14Z`
  * all at once: http://localhost:4301/0.1/activity?modifiedSince=2017-01-03T08%3A14%3A14Z&format=json-ld
  * Responses: if status codes are `304` No Modified or `405` Method Not Allowed the response body is empty. If the Response status is `200` it will return data specific to activites:

  ```turtle
  @base <http://data.hulib.helsinki.fi/attx/> .
  @prefix attxonto: <http://data.hulib.helsinki.fi/attx/onto#> .
  @prefix attx: <http://data.hulib.helsinki.fi/attx/> .
  @prefix owl: <http://www.w3.org/2002/07/owl#> .
  @prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
  @prefix xml: <http://www.w3.org/XML/1998/namespace> .
  @prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
  @prefix prov: <http://www.w3.org/ns/prov#> .
  @prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
  @prefix dcterms: <http://purl.org/dc/terms/> .
  @prefix sd: <http://www.w3.org/ns/sparql-service-description#> .
  @prefix pwo: <http://purl.org/spar/pwo/> .

  attx:work1
    a attxonto:Dataset, sd:Dataset ;
    dcterms:title "VIRTA Publication Dataset" ;
    dcterms:description "National list of publication that are part of the official VIRTA collection."@en;
    dcterms:publisher "Ministry of Culture and Education" ;
    dcterms:source <http://virta.fi/data_dump.csv> ; # actual source is a CSV file.
    dcterms:license <http://creativecommons.org/licenses/by/4.0/> .

  attx:work2
    a attxonto:Dataset, sd:Dataset ;
    dcterms:title "CRIS Publication Dataset" ;
    dcterms:description "National list of publication that are part of the official CRIS collection."@en;
    dcterms:publisher "Ministry of Culture and Education" ;
    dcterms:source <http://cris.fi/data_dump.csv> ; # actual source is a CSV file.
    dcterms:license <http://creativecommons.org/licenses/by/4.0/> .

  attx:work3
    a attxonto:Dataset, sd:Dataset ;
    dcterms:title "Selected VIRTA Publication Data in RDF" ;
    dcterms:description "Dataset includes titles and external identifiers for every publication and research fields, see mapping steps for details"@en;
    dcterms:license <http://creativecommons.org/licenses/by/4.0/> . # license should be the same as original data

  attx:activity2
    a prov:Activity , attxonto:WorkflowExecution ;
    prov:startedAtTime  "2016-11-17T13:02:10+02:00"^^xsd:dateTime ;
    prov:endedAtTime "2016-11-17T13:40:47+02:00"^^xsd:dateTime ;
    prov:qualifiedAssociation [
      a prov:Assocation ;
      prov:agent attx:ETL ;
      prov:hadPlan attx:wf2 ;
    ] ;
    prov:used attx:work1 ;
    prov:used attx:work2 ;
    prov:generated attx:work3 .

  # Specific Agent and Artifact information

  attx:ETL
    a prov:Agent ;
    attxonto:usesArtifact attx:UnifiedViews.

  attx:UnifiedViews
    a attxonto:Artifact ;
    dcterms:source <http://unifiedviews.eu> .

  ```
* `activity` and `workflow` POST APIs will return `405` as this operation current is not allowed.

## How the Information is Extracted

Extracting information associated to the provenance and workflow information can be classified in three types:
* Database information related to pipelines and pipeline execution;
* ATTX medata DPU plugin - see Example 2;
* Other DPU specific information - see Example 1 - such as SPARQL query or SPARQL endpoint.

Information specific to workflows and activities is extracted directly from the MySQL database for example activities and associated information are extracted from pipeline executions, however for information related to input and output graphs the attxMetadataPlugin DPU is used as illustrated in Example 2.

```sql
SELECT exec_pipeline.id AS 'activityId',
ppl_model.id AS 'workflowId',
exec_pipeline.t_start AS 'activityStart',
exec_pipeline.t_end AS 'activityEnd',
exec_pipeline.t_end AS 'lastChange'
FROM exec_pipeline, ppl_model
WHERE exec_pipeline.pipeline_id = ppl_model.id AND\
exec_pipeline.status = 5
ORDER BY ppl_model.id
```

As depicted in the examples below often html tags are encoded and even more so the tags within them are encoded a second time, thus making it difficult to extract the required information.

Example 1: SPARQL endpoint configuration
```xml
<object-stream>
  <MasterConfigObject>
    <configurations>
      <entry>
        <string>dpu_config</string>
        <string><object-stream>
  <eu.unifiedviews.plugins.extractor.sparqlendpoint.SparqlEndpointConfig__V1>
    &lt;endpoint&gt;http://dbpedia.org/sparql&lt;/endpoint&gt;
    &lt;query>PREFIX  dbo:  &amp;lt;http://dbpedia.org/ontology/&amp;gt;
PREFIX  dbp:  &amp;lt;http://dbpedia.org/property/&amp;gt;
PREFIX yago:	&amp;lt;http://dbpedia.org/class/yago/&amp;gt;
PREFIX fi:  &amp;lt;http://helsinki.fi/example&amp;gt;
CONSTRUCT {
           ?subject fi:hasName ?name.
           ?subject fi:hasOccupation ?occupation.
}
  WHERE
    {
      ?subject   a  dbo:Person;
      dbp:name  ?name ;
     dbo:occupation ?occupation.
     FILTER (regex(?name, &amp;quot;Stefan&amp;quot; ))
   }
Limit 100  &lt;/query&gt;
  </eu.unifiedviews.plugins.extractor.sparqlendpoint.SparqlEndpointConfig__V1>
</object-stream></string>
      </entry>
      <entry>
        <string>addon/faultToleranceWrap</string>
        <string><object-stream>
  <eu.unifiedviews.helpers.dpu.extension.faulttolerance.FaultTolerance_-Configuration__V1>
    <enabled>false</enabled>
    <exceptionNames class=&quot;linked-list&quot;/>
    <maxRetryCount>-1</maxRetryCount>
  </eu.unifiedviews.helpers.dpu.extension.faulttolerance.FaultTolerance_-Configuration__V1>
</object-stream></string>
      </entry>
    </configurations>
  </MasterConfigObject>
</object-stream>
```
Using the ATTX metadata plugin (see information at: [UnifiedViews plugins](Unified-Views-plugins.md))
Example 2: ATTX metadata configuration
```xml
<object-stream>
    <MasterConfigObject>
        <configurations>
            <entry>
                <string>dpu_config</string>
                <string><object-stream> <org.uh.attx.etl.uv.dpu.transformer.metadata.TransformerATTXMetadataConfig__V1>
&lt;inputGraphURI&gt;http://helsinki.fi/library/onto#dataset1&lt;/inputGraphURI&gt; &lt;inputGraphTitle&gt;Dbpedia&lt;/inputGraphTitle&gt;
                    &lt;inputGraphDescription&gt;Sparql Endpoint Dbpedia specific query&lt;/inputGraphDescription&gt; &lt;inputGraphPublisher&gt;Dbpedia&lt;/inputGraphPublisher&gt; &lt;inputGraphSource&gt;http://dbpedia.org/sparql&lt;/inputGraphSource&gt;
                    &lt;inputGraphLicence&gt;http://creativecommons.org/licenses/by-sa/3.0/&lt;/inputGraphLicence&gt; &lt;outputGraphURI&gt;http://helsinki.fi/library/onto#dataset2&lt;/outputGraphURI&gt; &lt;outputGraphTitle&gt;Testing specific output&lt;/outputGraphTitle&gt;
                    &lt;outputGraphDescription&gt;Test Data&lt;/outputGraphDescription&gt; &lt;outputGraphPublisher&gt;http://helsinki.fi/library&lt;/outputGraphPublisher&gt; &lt;outputGraphSource&gt;http://helsinki.fi/library&lt;/outputGraphSource&gt;
                    &lt;outputGraphLicence&gt;http://creativecommons.org/licenses/by-sa/3.0/&lt;/outputGraphLicence&gt; </org.uh.attx.etl.uv.dpu.transformer.metadata.TransformerATTXMetadataConfig__V1> </object-stream></string>
            </entry>
        </configurations>
    </MasterConfigObject>
</object-stream>
```

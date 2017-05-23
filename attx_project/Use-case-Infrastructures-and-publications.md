# UC1 - Infrastructures and Publications

This pages describes the use cases identified as UC1, which deals with combining publication and infrastructure data.
Steps:

1. Get publication, infrastructure metadata from UH with links between them.
2. Enrich infrastructure metadata using the service descriptions coming from the national infrastructure databank.
3. Profit?

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Data sources](#data-sources)
    - [Research infrastructures databank](#research-infrastructures-databank)
    - [TUHAT - University of Helsinki's CRIS system](#tuhat---university-of-helsinkis-cris-system)
  - [Workflows](#workflows)
  - [Working Data](#working-data)

<!-- TOC END -->

## Data sources

This section describes the the way different external datasets are accessed what the data looks like.

### Research infrastructures databank

* UI: http://infras.openscience.fi/
* API: http://avaa.tdata.fi/api/jsonws?contextPath=/tupa-portlet
* API documentation:http://avaa.tdata.fi/adata/tupa/
* Datamodel: http://iow.csc.fi/model/tipa/

### TUHAT - University of Helsinki's CRIS system

We are using the test instance of the TUHAT system.

* UI:https://tuha-test.it.helsinki.fi
* API: https://tuha-test.it.helsinki.fi/ws
* API documentation: https://tuha-test.it.helsinki.fi/ws/doc

Because API of the current PURE version does not support equipments (=infrastructure) we are forced to use other means to extract the information.

The end result will a Turtle file that conforms to the data model described on this page. The file is downloadable from the data.hulib.helsinki.fi server.

## Workflows

We will be using three workflows:

* `harvestFromInfrastructureDatabank`
 * Retrieves JSON from service API and transforms it into internal RDF (see below).
* `harvestFromHYCRIS`
 * Retrieves three RDF files from library's server and uploads them to the internal graph.
* `linking`
 * Creates identifier based links between infrastructure data coming from two different sources.

Workflows use very basic and custom built DPUs for transformations and processing.

* uc1-infras2internal - transformer
* uc1-linker - transformer
 * link processor that knows how to compare URN pid and URN urls (urn:nbn:fi:research-infras-201607252 vs. http://urn.fi/urn:nbn:fi:research-infras-201607252). PIDs are coming from infrastructure databank. There is no place in Tuhat to add external identifiers, but there is a place for links.

## Working Data

This model should evolve as the implementation of the use case becomes more complex.
We assume that TUHAT infrastructure data contains identifiers that point to the infrastructure bank. The idea is to enrich the infrastructure information.

See [Namespaces](Namespaces.md)

**Example data**

Infrastructure databank
```turtle
<http://data.hulib.helsinki.fi/attx/work/{workflowID}/infra/{urn}>
  a <http://data.hulib.helsinki.fi/attx/types/Infrastructure> ;
  dct:title "{name}" ;
  dct:identifier <{urn}> .
```

Tuhat
```turtle
<http://data.hulib.helsinki.fi/attx/work/{workflowID}/pub/{uuid}>
  a <http://data.hulib.helsinki.fi/attx/types/Publication> ;
  dct:title "{title}" .

<http://data.hulib.helsinki.fi/attx/work/{workflowID}/infra/{uuid}>
  a <http://data.hulib.helsinki.fi/attx/types/Infrastructure> ;
  dct:identifier "{uuid}" ;
  dct:title "{title}" ;
  attx:link <{links url}> .

<http://data.hulib.helsinki.fi/attx/work/{workflowID}/pub/{uuid}>
  dct:relation <http://data.hulib.helsinki.fi/attx/work/{workflowID}/infra/{uuid}> .
```

Link processor
```turtle
<http://data.hulib.helsinki.fi/attx/{workflowID}/work/infra/{urn}>
   skos:exactMatch <http://data.hulib.helsinki.fi/attx/{workflowID}/work/pub/{uuid}>
```

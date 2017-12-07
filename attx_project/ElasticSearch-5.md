# ElasticSearch 5.x

Version 5.x of ElasticSearch provides an alternative indexing endpoint for the distribution component exposed data, and the same time different data structure.

Another functionality is the centralised logging which aims to connect provenance information with the components processes.

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
  - [Data structure](#data-structure)
    - [Indexing Working data](#indexing-working-data)
    - [Indexing JSON-LD Provenance](#indexing-json-ld-provenance)

<!-- TOC END -->

## Data structure

Currently ATTX Semantic Broker offers functionality for indexing semantic data as plain JSON (formatted via a JSON-LD frame by using [Graph Framing Service](Service-Graph-Framing.md)) or a whole JSON-LD (using [Graph Framing Service](Service-Graph-Framing.md) but providing an empty frame; this might raise issues with the depth of the JSON documents https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#mapping-limit-settings that by default is `20`).

### Indexing Working data

Indexing data formatted via a JSON-LD frame, takes the complexity of formatting the data from the client, exposing only the necessary information for the client to use in a specific application.

Given a JSON-LD frame:
```json
{
  "@context": {
    "dc": "http://purl.org/dc/elements/1.1/",
    "ex": "http://example.org/vocab#",
    "dct": "http://purl.org/dc/terms/",
    "work": "http://data.hulib.helsinki.fi/attx/work#",
    "title": "dct:title"

  },
  "@type": "work:Publication",
  "@explicit": true,
  "@requireAll": false,
  "title": "",
}
```
The following results are available in ElasticSearch
```json
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 3,
        "max_score": 1.0,
        "hits": [{
            "_index": "testindex_service_20171207_120336",
            "_type": "testdoc4",
            "_id": "tutka:75547",
            "_score": 1.0,
            "_source": {
                "@id": "tutka:75547",
                "@type": "work:Publication",
                "title": "The Importance of Phytoplankton Biomolecule Availability for Secondary Production"
            }
        }, {
            "_index": "testindex_service_20171207_120336",
            "_type": "testdoc4",
            "_id": "tutka:68691",
            "_score": 1.0,
            "_source": {
                "@id": "tutka:68691",
                "@type": "work:Publication",
                "title": "A Competence-based View on the Global Software Development Process"
            }
        }, {
            "_index": "testindex_service_20171207_120336",
            "_type": "testdoc4",
            "_id": "tutka:69464",
            "_score": 1.0,
            "_source": {
                "@id": "tutka:69464",
                "@type": "work:Publication",
                "title": "A gap analysis of Internet-of-Things platforms"
            }
        }]
    }
}
```

### Indexing JSON-LD Provenance

Example of Provenance data indexed in ElasticSearch as JSON-LD.
Snapshot of a provenance document:

```json
{
    "_index": "attx_service_20171205_155508",
    "_type": "workflow4_activity4",
    "_id": "http://data.hulib.helsinki.fi/attx/workflow4_activity4_UnifiedViews",
    "_score": 1.0,
    "_source": {
        "http://data.hulib.helsinki.fi/attx/onto#hasStatus": "success",
        "http://purl.org/dc/terms/title": "Publish dataset Activity4",
        "@id": "http://data.hulib.helsinki.fi/attx/workflow4_activity4_UnifiedViews",
        "http://www.w3.org/ns/prov#qualifiedAssociation": {
            "@id": "http://data.hulib.helsinki.fi/attx/workflow4_activity4_association_9ca904254f6dd6f3ab04911d87fdc863",
            "@type": "http://www.w3.org/ns/prov#Association",
            "http://www.w3.org/ns/prov#hadPlan": {
                "http://www.w3.org/2000/01/rdf-schema#label": "Workflow: workflow4_activity4",
                "@id": "http://data.hulib.helsinki.fi/attx/workflow4_activity4",
                "@type": ["http://www.w3.org/ns/prov#Plan", "http://data.hulib.helsinki.fi/attx/onto#Workflow"]
            }
        },
        "http://www.w3.org/ns/prov#qualifiedGeneration": [{
            "http://www.w3.org/ns/prov#entity": {
                "@id": "http://data.hulib.helsinki.fi/attx/entity_58b5c907d4f9ea7d0b38815530267a39"
            },
            "@id": "http://data.hulib.helsinki.fi/attx/generated_73bb9267396ee27272353dfe12715a29",
            "@type": "http://www.w3.org/ns/prov#Generation"
        }, {
            "http://www.w3.org/ns/prov#entity": {
                "http://data.hulib.helsinki.fi/attx/title": "Published DS",
                "http://www.w3.org/2000/01/rdf-schema#comment": "Generated Dataset",
                "http://data.hulib.helsinki.fi/attx/uri": "testindex",
                "http://www.w3.org/2000/01/rdf-schema#label": "Published DS",
                "http://data.hulib.helsinki.fi/attx/description": "test",
                "http://purl.org/dc/terms/source": ["{u'uri': u'testindex'}", "{u'title': u'Published DS', u'uri': u'testindex', u'description': u'test'}", "testindex"],
                "@id": "http://data.hulib.helsinki.fi/attx/entity_58b5c907d4f9ea7d0b38815530267a39",
                "@type": ["http://www.w3.org/ns/prov#Entity", "http://data.hulib.helsinki.fi/attx/onto#Dataset"]
            },
            "@id": "http://data.hulib.helsinki.fi/attx/generated_9e49b97e924bcb33c7b6ef6f475f376c",
            "@type": "http://www.w3.org/ns/prov#Generation"
        }],
        "http://www.w3.org/ns/prov#qualifiedCommunication": null,
        "http://www.w3.org/ns/prov#qualifiedUsage": [{
            "http://www.w3.org/ns/prov#entity": {
                "@id": "http://data.hulib.helsinki.fi/attx/entity_e288657519292e97999ef572d18da25e"
            },
            "@id": "http://data.hulib.helsinki.fi/attx/used_7e09c07b247836ed5061d5970bdb8aaf",
            "@type": "http://www.w3.org/ns/prov#Usage"
        }, {
            "http://www.w3.org/ns/prov#entity": {
                "@id": "http://data.hulib.helsinki.fi/attx/entity_f1d141de65f33e5f8585e2e1320446c8"
            },
            "@id": "http://data.hulib.helsinki.fi/attx/used_bea1221ed9964ce238c80a23ce478797",
            "@type": "http://www.w3.org/ns/prov#Usage"
        }, {
            "http://www.w3.org/ns/prov#entity": {
                "@id": "http://data.hulib.helsinki.fi/attx/entity_0195a4f171e7f9a59fb63e5cdb276046"
            },
            "@id": "http://data.hulib.helsinki.fi/attx/used_754826bd75c25f278f76be731b61063a",
            "@type": "http://www.w3.org/ns/prov#Usage"
        }],
        "http://www.w3.org/ns/prov#generated": {
            "@id": "http://data.hulib.helsinki.fi/attx/entity_58b5c907d4f9ea7d0b38815530267a39"
        },
        "http://www.w3.org/ns/prov#endedAtTime": {
            "@type": "http://www.w3.org/2001/XMLSchema#dateTime",
            "@value": "2017-12-05T09:18:11"
        },
        "http://www.w3.org/ns/prov#wasAssociatedWith": {
            "http://www.w3.org/2000/01/rdf-schema#label": "UnifiedViews",
            "@id": "http://data.hulib.helsinki.fi/attx/UnifiedViews",
            "@type": ["http://data.hulib.helsinki.fi/attx/onto#Artifact", "http://www.w3.org/ns/prov#Agent"]
        },
        "http://www.w3.org/2000/01/rdf-schema#label": "Publish dataset Activity4",
        "@type": ["http://data.hulib.helsinki.fi/attx/onto#WorkflowExecution", "http://www.w3.org/ns/prov#Activity"],
        "http://www.w3.org/ns/prov#startedAtTime": {
            "@type": "http://www.w3.org/2001/XMLSchema#dateTime",
            "@value": "2017-12-05T09:18:01"
        },
        "http://www.w3.org/ns/prov#used": [{
            "http://purl.org/dc/terms/source": "http://data.hulib.helsinki.fi/attx/work/wf_2_step_10/output",
            "@id": "http://data.hulib.helsinki.fi/attx/entity_0195a4f171e7f9a59fb63e5cdb276046",
            "@type": "http://www.w3.org/ns/prov#Entity",
            "http://www.w3.org/2000/01/rdf-schema#comment": "Used Dataset"
        }, {
            "http://purl.org/dc/terms/source": "http://data.hulib.helsinki.fi/attx/work/wf_1_step_2/output",
            "@id": "http://data.hulib.helsinki.fi/attx/entity_e288657519292e97999ef572d18da25e",
            "@type": "http://www.w3.org/ns/prov#Entity",
            "http://www.w3.org/2000/01/rdf-schema#comment": "Used Dataset"
        }, {
            "http://purl.org/dc/terms/source": "http://data.hulib.helsinki.fi/attx/work/wf_3_step_14/output",
            "@id": "http://data.hulib.helsinki.fi/attx/entity_f1d141de65f33e5f8585e2e1320446c8",
            "@type": "http://www.w3.org/ns/prov#Entity",
            "http://www.w3.org/2000/01/rdf-schema#comment": "Used Dataset"
        }]
    }
}
```

# ElasticSearch 5.x

Version 5.x of ElasticSearch provides an alternative indexing endpoint for the distribution component exposed data, and the same time different data structure.

Another functionality is the centralised logging which aims to connect provenance information with the components processes.

## Data structure

Currently ATTX Semantic Broker offers functionality for indexing semantic data as plain JSON (formatted via a JSON-LD frame) or a whole JSON-LD.

### Elasticsearch 5.x and JSON derived from JSON-LD frame

Indexing data formatted via a JSON-LD frame, takes the complexity of formatting the data from the client, exposing only the necessary information for the client to use in a specific application.

Given a JSON-LD frame (included in the POST `index` request):
```json
{
  "@context": {
    "@vocab": "http://data.hulib.helsinki.fi/attx/",
    "dcterms": "http://purl.org/dc/terms/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "attx-types": "http://data.hulib.helsinki.fi/attx/types/",
    "attx-types:Infrastructure": {
    "@type": "@id"
  },
    "attx:hasclusteredID": {
      "@id": "http://data.hulib.helsinki.fi/attx/id"
    }
    ,
    "dcterms:identifier": {
      "@type": "@id",
      "@id": "http://purl.org/dc/terms/identifier"
    }
  },
  "@type": "attx-types:Infrastructure"
}
```
The resulting JSON-LD would have the following structure:
```json
{
  "@context": {
    "@vocab": "http://data.hulib.helsinki.fi/attx/",
    "dcterms": "http://purl.org/dc/terms/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "attx-types": "http://data.hulib.helsinki.fi/attx/types/",
    "attx-types:Infrastructure": {
      "@type": "@id"
    },
    "attx:hasclusteredID": {
      "@id": "http://data.hulib.helsinki.fi/attx/id"
    },
    "dcterms:identifier": {
      "@type": "@id",
      "@id": "http://purl.org/dc/terms/identifier"
    }
  },
  "@graph": [
    {
      "@id": "http://data.hulib.helsinki.fi/attx/3/work/infra/urn:nbn:fi:research-infras-201607251",
      "@type": "attx-types:Infrastructure",
      "attx:hasclusteredID": {
        "@id": "urn:nbn:fi:research-infras-201607251"
      },
      "dcterms:identifier": "urn:nbn:fi:research-infras-201607251",
      "dcterms:title": "European Social Survey"
    },
    {
      "@id": "http://data.hulib.helsinki.fi/attx/3/work/infra/urn:nbn:fi:research-infras-2016072510",
      "@type": "attx-types:Infrastructure",
      "attx:hasclusteredID": {
        "@id": "urn:nbn:fi:research-infras-2016072510"
      },
      "dcterms:identifier": "urn:nbn:fi:research-infras-2016072510",
      "dcterms:title": "Finnish Marine Research Infrastructure"
    }]
  }
```

### Elasticsearch 5.x indexing JSON-LD

Provides more flexibility on the client side for formatting the data, and at the same adding more complexity to the client requests.

To Be Continued.

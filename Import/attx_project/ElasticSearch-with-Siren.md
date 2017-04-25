# ElasticSearch with Siren

This page describes the components and configuration needed to get the ATTX ES plugin up and running. This information should be used as the basis for creating a containerized version of the API component

**Siren plugin only supports ES up till version 1.3.4.**

### Table of Contents

* [Mapping](#mapping)
* [Plugins](#plugins)
  * [siren-elasticsearch 1.4](#siren-elasticsearch-14)
  * [analysis-icu 1.3.4](#analysis-icu-134)
  * [attx-es-api](#attx-es-api)

## Mapping

In addition to the plugins listed below, the system requires that indexes are created with the following mapping.

```{json}
{"_default_":
  {"properties":
    {"_siren_source":
      {"analyzer": "concise",
       "postings_format": "Siren10AFor",
       "store": "no",
       "type": "string"
      }
    },
    "_siren": {},
    "dynamic_templates": [
      {"rawstring":
        {"match": "*",
         "match_mapping_type": "string",
         "mapping": {
           "type": "string",
           "index": "analyzed",
           "fields": {
             "{name}_raw": {
               "type": "string",
               "analyzer": "case_insensitive_sort"
             }
           }
         }
       }
      }
    ]
  }
}
```

## Plugins

These are the required plugins:

* [siren-elasticsearch 1.4](#siren-elasticsearch-14)
* [analysis-icu 1.3.4](#analysis-icu-134)
* [attx-es-api](#attx-es-api)

### siren-elasticsearch 1.4

[https://github.com/sirensolutions/siren](https://github.com/sirensolutions/siren)

Plugin that allows one to index and query nested documents easily and efficiently.  
Unfortunately, it is not maintained any more and we should come up with a replacement.

Installation:

```
bin/plugin --install com.sindicetech.siren/siren-elasticsearch/1.4
```

Configuration:

Add the following to the elasticsearch.yml

```{yml}
# Settings for the SIREn Plugin
siren:
  analysis:
    # Enabled the use of wildcard for node attribute - Only for the concise mode This will increase the index size,
    # as it will expand terms at indexing time. It is recommended to deactivate it if one does not need attribute
    # wildcards.
    concise.attribute_wildcard.enabled: true
    # Mappings between datatype and analyzers. The analyzers must be referenced by their registered logical name.
    datatype:
      http://www.w3.org/2001/XMLSchema#string:
        index_analyzer: standard
        search_analyzer: lowerWhitespaceAnalyzer # here we refer to a custom analyzer that is below
      http://json.org/field:
        index_analyzer: whitespace
      http://www.w3.org/2001/XMLSchema#double:
          index_analyzer: double
      http://www.w3.org/2001/XMLSchema#long:
          index_analyzer: long
      http://www.w3.org/2001/XMLSchema#boolean:
          index_analyzer: whitespace
      uri: # for NCPR demo
          index_analyzer: whitespace
    qname:
        xsd : http://www.w3.org/2001/XMLSchema#
        json: http://json.org/
```

### analysis-icu 1.3.4

[https://github.com/elastic/elasticsearch-analysis-icu](https://github.com/elastic/elasticsearch-analysis-icu)

Extended Unicode support for ES. Needed to get the sorting working. Version 2.3.0 works with ES 1.3.4.

Installation:

```
bin/plugin install elasticsearch/elasticsearch-analysis-icu/2.3.0
```

Configuration:

```{yml}
index:
    analysis:
        analyzer:
            lowerWhitespaceAnalyzer:
                type: custom
                tokenizer: whitespace
                filter: [lowercase]

            case_insensitive_sort:
                tokenizer: keyword
                filter: [ "icu_collation" ]
```

### attx-es-api

Plugin that provides simple query interface and then transforms those queries into ones that can be executed by the siren plugin.

Installation:

Our plugin can be installed in the same manner as the other ones. [https://www.elastic.co/guide/en/elasticsearch/reference/1.4/modules-plugins.html](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/modules-plugins.html)

Releases can installed from github using the following command:

```
bin/plugin --install <org>/<user/component>/<version>
```

dev version can use the file based installation method:

```
bin/plugin --url file:///path/to/plugin --install plugin-name
```

**How it works**

Simple queries like "test" are translated into simple NodeQuery.

_TODO: continue this!_

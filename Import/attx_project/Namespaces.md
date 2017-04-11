# ATTX Namespaces

Base namespaces and named graphs:

* `http://data.hulib.helsinki.fi/attx/onto#` - ontology and associated classes
* `http://data.hulib.helsinki.fi/attx` - to be used as a base for data
* `http://data.hulib.helsinki.fi/attx/prov` - for provenance data
* `http://data.hulib.helsinki.fi/attx/ids` - for clusted data ids
* `http://data.hulib.helsinki.fi/attx/{workflowID}/work` - for working data
  * `http://data.hulib.helsinki.fi/attx/work/{workflowID}/infra` - for infrastructure
  * `http://data.hulib.helsinki.fi/attx/{workflowID}/{workflowID}/work/pub` - for  publication
* `http://data.hulib.helsinki.fi/attx/id` - service whatever comes after the id should resolve the id

The [Graph Manager API](Graph-Manager-API.md) generates working datasets by following the `http://data.hulib.helsinki.fi/attx/work{generatedID}` where the `generatedID` is a result of hash algorithm. To be pointed out that the `http://data.hulib.helsinki.fi/attx/work` is the recommended base for the working datasets.

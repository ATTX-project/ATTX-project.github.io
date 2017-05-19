# Graph component

The ATTX Graph component associated to this project has the main goal of aggregating the data that flows within the platform, types of transformations \(and associated workflows\), the provenance information \(agent and ETL processes performed\) and other meta data.

Graph Component is designed to:

* retrieve information from the [Workflow API](Workflow-API.md) in order to update the **Provenance and Workflow Data** graphs;
* manage the mapping - RML mappings;
* source data queries - select specific data from the [Graph Store](Graph-Store.md) and after applying the mapping deliver it to target output;
* target output \(e.g. endpoint API\) - where to output the [Graph Store](Graph-Store.md) data.
* There is an [Graph Manager API](Graph-Manager-API.md) which handles all these specific tasks
* Ontology based linking/clustering of IDs

**Github Repository: **[**graph-component**](https://github.com/ATTX-project/graph-component)

The Workflow Component consists of following parts:

* [Graph Manager API](Graph-Manager-API.md)
  * [Graph Store](Graph-Store.md)
    * [Ontology/Data model](Data-Model.md)
    * [Namespaces](Namespaces.md)
    * [Linking](Linking-graphs.md)

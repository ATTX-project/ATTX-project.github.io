# Graph component

The ATTX Graph component associated to this project has the main goal of aggregating the data that flows within the platform, types of transformations \(and associated workflows\), the provenance information \(agent and ETL processes performed\) and other meta data.

Graph Component services are designed to:
* source data queries - select specific data from the [Graph Store](Graph-Store.md) and after applying the mapping deliver it to target output;
* target output (e.g. endpoint API) - where to output the [Graph Store](Graph-Store.md) data.
* There is an [Graph Manager Service](Service-Graph-Manager.md) which handles all these specific tasks
* Ontology based linking/clustering of IDs

The Workflow Component consists of following parts:

* [Graph Manager Service](Service-Graph-Manager.md)
    * [Graph Manager Message Examples](Examples-Graph-Manager-Service.md)
  * [Graph Store](Graph-Store.md)
    * [Ontology/Data model](ATTX-Data-Model.md)
* [Linking Strategies](Linking-Strategies.md)
* [Provenance Service](Service-Provenance.md)
* [RML Service](Service-RML.md)

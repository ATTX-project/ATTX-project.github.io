# Distribution Component

This ATTX Distribution Component provides the interface between the Graph Component and the applications designed for consuming and disseminating data.

**Github Repository: **[**distribution-component**](https://github.com/ATTX-project/distribution-component)

The Distribution Component is composed of:

* [ElasticSearch with Siren](ElasticSearch-with-Siren.md) artifact which has the role of indexing and making the Graph Store knowledge convenient accessible via an API;
* ElasticSearch 5.2.0 which provides the latest functionality in order to index data as plain JSON \(after applying a JSON-LD frame\), JSON-LD or capturing logs.

More on data indexing see: [Graph-Manager-API](Graph-Manager-API.md)

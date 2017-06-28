## ETL Artifacts

The Workflow Component ETL artifact consists of three core components as depicted in Figure 1:

* Ingestion
* Processing
* Distribution

Other ETL components could be added to the ETL stack with the purpose of enhancing the results. Examples include:

* NER for extracting Places, Organization etc.
* Data converters

Some of the uses of the ETL workflow include: inferencing, calling and updating distribution endpoints, processing and transformation, updating ElasticSearch data and also calling functionalities from Graph Component, more precisely [Graph Manager API](Graph-Manager-API.md) which in return will make use of the services available.

A simplified version of the communication and data flow in the Semantic Broker is provided in Figure 1. For examples on specific services please refer to [ATTX Architecture Overview](ATTX-Architecture-Overview.md).

![Figure 1. Semantic Broker Component and Data Flow](images/etl_componentworkflow.svg)

Figure 1.  Semantic Broker Component and Data Flow

## Comparison of ETL Artifacts

The table below shows a basic comparison of some of the ETL tools and required functionalities with relation to the ATTX project.

| Tool | Workflows | Activities | REST API | Plugins | UI | License |
| --- | :---: | :---: | :---: | :---: | :---: | :---: |
| [Wings](https://github.com/IKCAP/wings) | Yes | Yes ? | No ? | No ? | Yes | Apache 2.0 |
| [LinkedPipes](https://github.com/linkedpipes/etl) | Yes | Yes | Yes | Yes ? | Yes | MIT |
| [DSwarm](http://www.dswarm.org/) | ? Maybe Transformations | ? Maybe Transformations | Yes | No ? | Yes | Apache 2.0 |
| [Web-Karma](https://github.com/usc-isi-i2/Web-Karma) | No, although there is Batch Mode | No, although there is Batch Mode | Yes | Yes, kinda | Yes | Apache 2.0 |
| [UnifiedViews](https://github.com/UnifiedViews/Core) | Yes | Yes | Yes \(Limited\) | Yes | Yes | GPL 3.0 |
| [FluidOps](http://www.fluidops.com/) | ? | ? | ? | Yes ? | Yes | Commercial |
| [Silk Framework](http://silkframework.org) | Yes, Tasks | Yes, Workspace | Yes | Yes | Yes | Apache 2.0 |
| [Pentaho](http://community.pentaho.com/projects/data-integration/) | Maybe ? via Jobs | Maybe ? via Jobs | No ? | Yes | Yes | Apache 2.0 |

## References

* [https://www.opensemanticsearch.org/etl](https://www.opensemanticsearch.org/etl)
* [https://github.com/pawl/awesome-etl\#python](https://github.com/pawl/awesome-etl#python)

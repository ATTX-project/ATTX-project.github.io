<h1 style="color:red">DRAFT - work in progress</h1>

# Use case Metax

Metax project aims to transform the data storage layer of the research data related services such as [Etsin](https://etsin.avointiede.fi/) and IDA by creating a centralized data service that is used by all the related services. Migration of existing solutions to the new centralized model also requires migration of data. ATT initiative has created a new model for research data catalogs, which is maintained in IOW service (http://iow.csc.fi/model/att/). The current idea is that the new model should also form the basis for the data managed in Metax, and therefore existing data must be converted to the new ATT model as part of the migration process.

## Approach

The goal of the ATTX Metax use case is to test and analyse the potential mappings from existing data to the new ATT model and the status and suitability of the new ATT model itself through prototyping. These prototypes will be implemented using ATTX components and configurations. Results of the use case should be beneficial both to the working group that is responsible for developing the model as well as to the project group in charge of implementing the migration of the existing data.

Implementation of this use case might be affected by the progress and schedule of the Metax project itself. For example if Metax project reaches the data migration phase before ATTX implementation, it might be good to change the plan for the use case or drop it completely.

## Deliverables

List of initial deliverables:

* ATTX feature that allows users to make complicated mappings. Mapping key-value type data records from Etsin service to the linked data based ATT model with multiple identifiable resources/records requires more complex tools than the other use cases.
* ATTX feature, which allows efficient distribution of RDF data. This can be implemented for example using LinkedDataFragments.
* ATTX feature, which validates selected data using data model description available from the IOW service. Descriptions can be for example JSON schemas or SHACL documents.
* Documention of mapping from Etsin (and IDA?) data to the new ATT model.
* PoC level implementation of Etsin (and IDA) data mapping to the new ATT model.


## Mappings

## Implementation

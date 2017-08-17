<h1 style="color:red">DRAFT - work in progress</h1>

# Mildred use case

The goal of the use case is to create a dataset that provides an aggregated view to the research data output of the University of Helsinki. This new dataset and accompanied data API are then used as the backend service for university's [Think Open](https://www.helsinki.fi/en/research/think-open) site. Think open site brings together and to promotes openness related activities, such as open research data, open source code and publications, within the University of Helsinki.

Use case is part of the [Mildred project](http://blogs.helsinki.fi/mildred/), which aims to update the research data infstructure provided by the University of Helsinki. Mildred consists of five subprojects that concentrate on different phases of the research process. ATTX co-operates with the subproject which is responsible for building and data publishing and metadata services.

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Approach](#approach)
  - [User stories](#user-stories)
  - [Data sources](#data-sources)
    - [Etsin](#etsin)
    - [B2Share](#b2share)
    - [Zenodo](#zenodo)
    - [Finto](#finto)
    - [IOW](#iow)
  - [Implementation](#implementation)

<!-- TOC END -->


## Approach

Dataset related metadata is still relatively sparse, compared to other research outputs, such as publications. In this use case we are concentrating solely on data sources that were chosen by the project Mildred's steering group.

One of the main ideas behind the use case, it the added value that ATTX broker deliver by running customizable validation and data quality processes over the harvested data. This additional data can also be published and used by the ThinkOpen site to deliver custom quality metrics alongside the dataset metadata.

Etsin is already a service that aggregates dataset metadata from different sources, so why make another similar dataset? Mildred use case is interested specifically in UH's research outputs whereas Etsin is a national service. By creating an UH specific dataset, we have control over both the content and structure of the dataset, which makes it easier to integrate it with ThinkOpen work or any other UH specific use. It is also possible that some day the data might flow in an opposite direction between UH's dataset and Etsin.

When it comes to identifying which datasets are somehow related to the University of Helsinki, we will start with the simple methods that should have good precision but might not have acceptable recall, in order to get **some** data. It is however possible to extend to those methods with more complex internal processing.

TODO: Link to OpenScienceFair poster draft.

## User stories

## Data sources

### Etsin

Etsin is a national research dataset registry that contains both manually inputted and harvested metadata.
Based on CKAN.

#### APIs
Documentation: http://openscience.fi/etsin-api
Data model: http://openscience.fi/etsin-data-model

**REST**

https://etsin.avointiede.fi/api/3

CKAN documentation: http://docs.ckan.org/en/ckan-2.7.0/api/#get-able-api-functions

**OAI-PMH**

https://etsin.avointiede.fi/oai?verb=Identify

Owner organization is only references in the header part of the record. For example:
https://etsin.avointiede.fi/oai?verb=GetRecord&identifier=urn:nbn:fi:csc-kata20170611202023598902&metadataPrefix=oai_dc

#### Usage

Every record in Etsin has an owner_org property, which connects it to the owning organization. Organizations (https://etsin.avointiede.fi/api/3/action/organization_list) form a hierarchy (https://etsin.avointiede.fi/organization).

**TODO: Is it possible to utilize the organizational hierarchy with the search API?**

Request https://etsin.avointiede.fi/api/3/action/package_search?&q=owner_org:01901 only return 10 hits that are explicitly connected with UH and not the 7000+ other records that are owned by UH related organizational units.

Getting all the UH related datasets using REST api can be implemented by first constructing a list/graph of UH and related organizations and then querying all of them. More detailed information about an organization can be found from /organization_show endpoint. For example.  https://etsin.avointiede.fi/api/3/action/organization_show?id=35367e6f-6cd3-40e4-b2fe-aa391d34eeef, where value of the id parameter can be either name or id of the organization. "groups" property of the returned document contains information about the parent groups, which can be used to create a graph that represents the organizational hierarchy. After harvesting the child->parent relationships, inverse links can be added using simple ontology processing.



### B2Share

[B2Share](https://b2share.eudat.eu/) is part of the [EUDAT](https://eudat.eu/) collaborative data infrastructure.

In B2Share the data is organized around communities, which all maintain their own metadata schemas for community related records. This means that harvested data structures can vary depending on which community the record belongs to. There is also a common schema that is shared with all the communities.

Based on Invenio

#### APIs

https://b2share.eudat.eu/api/

Documentation

* https://b2share.eudat.eu/help/api
* https://www.eudat.eu/services/userdoc/the-b2share-http-rest-api

Elastic based data structure.
B2Share also has OAI-PMH interface, but it seems that it has not been exposed yet.

#### Usage

There is no UH community, although Aalto has one, but UH related dataset are distributed amongst the research infrastructure and field specific communities. Affiliation related information is in the publisher and description fields.

Communities:

https://b2share.eudat.eu/api/communities/

Records by community:

https://b2share.eudat.eu/api/records/?q=community:867c4e67-9227-4b6f-8595-c97d37e9de61


### Zenodo

[Zenodo](https://zenodo.org) is a general purpose repository for all kinds of research outputs ranging from presentations to datasets. Users can create communities to organize content into subrepositories.

Based on Invenio

#### API

Documentation:
http://developers.zenodo.org/

**REST (beta - no full documentation yet 08-2017)**

https://zenodo.org/api

Zenode REST API most likely uses Elastic as their implementations, because the output format is identical to one returned by Elastic's aggregation queries.

**OAI-PMH**

https://zenodo.org/oai2d

**Output formats**
* datacite
* datacite3.

Both have OAI specific variants available, which encapsulate the same record content with OAI elements. These representations include fields such as <isReferenceQuality> and other additional (to datacite format) info. oai_datacite is the recommended metadata format.
Example output from Hulib community in DataCite format:
https://zenodo.org/oai2d?verb=ListRecords&metadataPrefix=oai_datacite&set=user-hulib

http://developers.zenodo.org/#metadata-formats


#### Usage

Affilations can be used to identify creators that are from UH. However, there are different variations:

```
<creator>
  <creatorName>X, Y</creatorName>
  <affiliation>Helsinki University Library</affiliation>
</creator>

<creator>
  <creatorName>X, Y, Helsinki University Library</creatorName>
</creator>

<creator>
  <creatorName>Helsinki University Library</creatorName>
</creator>
```

Output from OAI-PMH interface does not include links to the file metadata (links.bucket). OAI-PMH also has license information in URI format.

```
license: {
id: "CC-BY-4.0"
}
```

compared to

```
<rightsList>
  <rights rightsURI="https://creativecommons.org/licenses/by/4.0/" >Creative Commons Attribution 4.0</rights>
  <rights rightsURI="info:eu-repo/semantics/openAccess" >Open Access</rights>
</rightsList>
```

Harvesting is available using OAI-PMH protocol. Harvesting can be targeted using sets that correspond to communities. For example https://zenodo.org/oai2d?verb=ListRecords&set=user-hulib&metadataPrefix=oai_dc would return records published by the University of Helsinki library.

**TODO: What are the other UH related Zenode communities?**

We can create another dataset that links communities to UH and use that to make the first broad classification of records.

### Finto

[Finto](http://finto.fi/en/) or Finnish thesaurus and ontology service can be used to access up-to-date versions of maintained linked vocabularies, which can be used to describe datasets or linked to existing dataset describtion to provide linked data for complex queries and automatic inferencing of new data.

### IOW

[IOW](https://iow.csc.fi/) is service for creating and maintaining descriptions for interoperability. In the context of of this use case, it means that IOW can be used as the data source for schemas that allow for automatic validation of incoming or outgoing broker data.

## Implementation

Architectural overview:

![Architectural overview](images/ATTX-Mildred-architecture.svg)

Features:
* DataCite output format
* Custom output format if required
* Data validation based on JSON schemas
* simple data quality analysis
* Identifying dataset related to UH
* Deduplication of dataset metadata available from multiple sources

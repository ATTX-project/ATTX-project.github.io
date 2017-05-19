# Use case Jyväskylä - Parallel publishing dashboard

Parallel publication (rinnakkaistallennus in Finnish) is a version of the published research article that is freely available from the organizations digital repository or similar service. For more detailed description about parallel publishing and how it relates to OA in general see https://www.jyu.fi/tutkimus/rinnakkaisjulkaiseminen/en/open-access/parallel-publishing (Jyväskylä university).

The idea behind parallel publishing dashboard is to create a up-to-date data set and accompanying web application, which together allow users to browse and visualize current state of parallel publishing in Finland along different attributes, such as organization, field of science or any other custom classification scheme. Data set and application should be flexible and support multiple "views" against the same data.

University of Jyväskylä library has been doing similar work and results were presented by Pekka Olsbo as part of closing seminar for the Surima-project. His presentation slides can be downloaded from http://urn.fi/URN:NBN:fi-fe201701251319.

Our goal, as part of the ATTX project, is to build and setup working prototype of a data broker that could be used to drive a national open data service, but putting one in production is out of the scope of the project.

One could argue that [VIRTA](https://confluence.csc.fi/display/VIR/VIRTA-julkaisutietopalvelu) service already contains the same information and works on a national level with excellent coverage. However, ATTX use case can be seen as complementary to the VIRTA data set, because it provides a platform for new data aggregation and generation methods (e.g. data mining from parallel published versions, licensing permitting) and allows for alternative "views" on parallel publishing (e.g. grouping based on custom classification).  

Also, VIRTA is the official publication data set with strict requirements for completeness and quality, where as the data set created in this use case has less stringent expectations for incoming data and is therefore not designed to work as a basis for any official reporting.

## Approach

In order to able to works with percentages, we need metadata data for every publication and well as the parallel published subset of publications. The the approach is to use combination of harvested (VIRTA) and master (CRIS) data sources in order to create the data set for all publications. Then to use repositories, institutional, national and subject based, to get metadata data about the parallel published versions and associated files. Simple linking between these two data sets is done using the existing identifiers, such as DOIs and URNs.


## User stories

* As an admin, I want to create the initial data data from one of the publication metadata sources.

* As an admin, I want to setup incremental harvesting of updates to the already existing data set, which should contain only changes made after the latest successful harvesting.

* As an admin, I want to add new properties to the existing data publication metadata data set.

* As an admin, I want add new vocabulary for classifying resources in the existing data set. Classification is based on the existing publication metadata i.e. it is created by linking set existing metadata values e.g. field of science to new concept.
 * I want to make sure that the vocabulary data is updated every month
 * I want to create a link from publication to the new vocabulary

* As an admin, I want to publish publication metadata so that is can be queried using the new vocabulary

* As an interested anonymous user, I want to be able to see the current parallel publishing situation

* As a user, I want be able to group data set according different vocabulary, e.g. field of science, organization or year.

* As a user, I want to see how the parallel publishing situation has evolved between years X and Y given group Z.
For example, percentage of parallel published articles between 2015-2017 in the field of physics.

* As a user, I want to download the full data dump of the broker in CSV format.

## Data sources

The use case is built upon publication and file metadata.

### VIRTA

National publication metadata registry with excellent coverage and regularly updated content would be a very good data source for base line publication metadata.

[Documentation](https://confluence.csc.fi/pages/viewpage.action?pageId=65922061) about the data harvested by VIRTA.

2017 harvested data includes link to the parallel published version, but this information is missing from 2015 model and less strict in 2016, so there is room for ATTX type of broker to fill in the blanks.

### CRIS systems

Universities' [CRIS systems](https://en.wikipedia.org/wiki/Current_research_information_system) are usually the master sources for publication related data. If there is a need for data that is not part of the VIRTA harvested data, it should be possible to get from the CRIS system. CRIS harvesting might also be beneficial for organizations that still submitting their data to VIRTA only once(?) a year, but are still able to make more recent data available for access.

### Repositories



## Required features

History data available
Incremental harvesting
configurable CSV to RDF transformation
configurable XML to RDF tranformation
publishing data set as a CSV file
publishing data set as queryable REST API
identifier based linking
generating new data based on ontologies

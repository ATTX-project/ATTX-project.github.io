This user guide is for people who have ATTX platform instance up and running and want to start working with data. 

# ATTX DPUs 
ATTX platform ships with custom DPUs that must be used when designing pipelines. 

## Metadata 
This plugin defined the metadata for pipeline's source and target datasets. 

Source dataset can be external or internal. In case of an external data, user must add name, description, publisher and license for the dataset.  For internal datasets, i.e. data that is already part of the platform's graph, user only needs to selected the desired URI of the working graph. User can select multiple internal input graphs, but only one external (why?).

Target dataset always requires name and description information.  

## Mapper 

** This one is still missing ** 

Used to input/generate configurations for RML based mapping implementation.

## Linker

**Maybe adding provenance generation to the GMAPI was not a good idea**

This plugin is used to select and configure platforms different linking strategies. Input graph information is added by connecting DPU's inputGraphs edge to the Metadata DPU. 

Finally Linker's output should be connected to Uploader DPU. 

### Linking strategies

**Identifier based linking**

Uses properties from the input datasets that have been classified as identifiers and tries to find exact matches.

## Uploader 

Uploader is used to add the resulting RDF dataset to the internal graph. It has two required inputs: rdf data and target graph. RDF data should be connected from any DPU that generates the final version of the data, for example Linker, RDF merger or RMLMapper DPUs. Target graph information is added by connecting Uploader to the Metadata DPU.  

# UV DPUs
UnifiedViews comes with many nice DPUs that allow users to do basic functions such as downloading files and interacting with REST APIs. 

## uv-e-httpRequest


# Example

Linking Helda thesis data and FINTO subject heading data. This example adds multilingual labels for thesis related subject headings. 



**Helda thesis**

Dataset includes basic bibliographic information about theses made in the university of Helsinki. One of the fields is dc.subject.YSO 

https://ethesis.helsinki.fi/repository/handle/123456789/6350?show=full

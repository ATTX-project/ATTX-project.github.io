## Use Cases

* Jyväskylä partnership
* Hanken partnership

## Visualizations

* https://github.com/Data2Semantics/provoviz
* http://www.ontodia.org/
* https://github.com/ktym/d3sparql
* http://mgskjaeveland.github.io/sgvizler/
* http://rc.lodac.nii.ac.jp/rdf4u/
* https://github.com/alangrafu/visualRDF

We can take inspiration in the visualisation from such a platform: https://github.com/refinery-platform/refinery-platform

short poster: Interactive Visualization of Provenance Graphs for Reproducible Biomedical Research http://gehlenborg.com/wp-content/uploads/refinery_poster_biovis-2015.pdf
or: SATORI: A System for Ontology-Guided Visual Exploration of Biomedical Data Repositories http://www.biorxiv.org/content/biorxiv/early/2016/04/05/046755.full.pdf

or more visualisation examples:
* https://flekschas.github.io/d3-list-graph/
* https://github.com/Data2Semantics/provoviz
* http://www.caleydo.org/publications/2016_eurovis_avocado/ with: http://demo.caleydo.org/pathfinder/
* https://github.com/twitter/ambrose
* https://github.com/neerajdembla212/d3-snake-chart
* http://square.github.io/crossfilter/ - multidimensional filtering

## Ideas

*  we integrate data via a workflow, for example from 3 sources, at the end an entity which combines all the information from the 3 data sources. At the end we do a quality evaluation of the provenance which provided a larger quantity of information to the result, assuming that the entity should be `skos:exactMatch` in all 3 datasets. Because we can tell people what is missing in their data.  It can also be a mechanism to tell we can link or not your data.
* grouping different editions together
* URI dereferencing and reconciliation consistency

# Project related data sources

A list of data sets and APIs relevant to the project.
* https://finto.fi/en/ - specifically https://finto.fi/okm-tieteenala/fi/
* [UC1: Infrastructures and publications](https://github.com/ATTX-project/project-management/wiki/UC1---Infrastructures-and-publications) - data for Use Case 1 which handle Infrastructures and publications

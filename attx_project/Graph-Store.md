# Graph Store

<!-- TOC START min:1 max:3 link:true update:true -->
  - [Configuring the Graph Store](#configuring-the-graph-store)
    - [Configuring Fuseki](#configuring-fuseki)
    - [Querying the Default Graph](#querying-the-default-graph)
    - [Adding Reasoning Capability](#adding-reasoning-capability)

<!-- TOC END -->

Graph Store functionalities include:
* managing up to date **Working Data** and **Provenance and Workflow Data** graphs;
* associates  **Working Data** with corresponding **Provenance and Workflow Data**;
* base graph store image is provided with ATTX specific **Provenance and Workflow Ontology**;
* is the LODSTOCK - Linked Open Data Stack for the platform generated data.

Current store implementation: [Apache Fuseki](https://jena.apache.org/documentation/fuseki2/index.html)

## Configuring the Graph Store

### Configuring Fuseki

```turtle
@prefix :        <#> .
@prefix fuseki:  <http://jena.apache.org/fuseki#> .
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:    <http://www.w3.org/2000/01/rdf-schema#> .
@prefix tdb:     <http://jena.hpl.hp.com/2008/tdb#> .
@prefix ja:      <http://jena.hpl.hp.com/2005/11/Assembler#> .
@prefix dc:      <http://purl.org/dc/elements/1.1/> .

[] rdf:type fuseki:Server ;
   fuseki:services (
     <#service_test>
     <#service_prod>
   ) .

# TDB loader configuration
[] ja:loadClass "org.apache.jena.tdb.TDB" .
tdb:DatasetTDB  rdfs:subClassOf  ja:RDFDataset .
tdb:GraphTDB    rdfs:subClassOf  ja:Model .

# Development service endpoint for Use-case Data.
# Recommended would be to split this into 2 services one for query with the inferenced dataset and one for update.
<#service_prod> rdf:type fuseki:Service ;
  rdfs:label                        "TDB Service" ;
  fuseki:name                       "ds" ;       # http://host:port/ds
  fuseki:serviceQuery               "sparql" ;   # SPARQL query service
  fuseki:serviceQuery               "query" ;    # SPARQL query service (alt name)
  fuseki:serviceUpdate              "update" ;   # SPARQL update service
  fuseki:serviceUpload              "upload" ;   # Non-SPARQL upload service
  fuseki:serviceReadWriteGraphStore "data" ;     # SPARQL Graph store protocol (read and write)
  # A separate read-only graph store endpoint:
  fuseki:serviceReadGraphStore      "get" ;      # SPARQL Graph store protocol (read only)
  fuseki:dataset                    <#dataset> ;
  .

  #  http://data.hulib.helsinki.fi/attx - to be used as a base for data
  #  http://data.hulib.helsinki.fi/attx/prov - for provenance data
  #  http://data.hulib.helsinki.fi/attx/ids - for clusted data ids

<#dataset> rdf:type ja:RDFDataset ;
  ja:defaultGraph <#default> ;
  # The graphs below will not be persistent in terms of namespacing.
  # problem pointed http://mail-archives.apache.org/mod_mbox/jena-users/201603.mbox/%3CCA%2BRbaMCSPHOO6%3DHTBETY-C1e2VfDEhHRfJ0GMSBfwQ70KEsm3Q%40mail.gmail.com%3E
  ja:namedGraph
      [ ja:graphName      <http://data.hulib.helsinki.fi/attx/onto> ;
        ja:graph          <#attx-ontodb> ] ;
  ja:namedGraph
      [ ja:graphName      <http://data.hulib.helsinki.fi/attx/ids> ;
        ja:graph          <#attx-ids> ] ;
  ja:namedGraph
      [ ja:graphName      <http://data.hulib.helsinki.fi/attx/prov> ;
        ja:graph          <#attx-prov> ] ;
  .

<#default> rdf:type tdb:GraphTDB ;
  tdb:dataset <#tdb_prod_dataset> ;
  .

<#attx-ontodb> rdf:type ja:OntModel ;
  ja:initialContent <#attx-onto> ;
  .

<#attx-ids> rdf:type tdb:GraphTDB ;
  tdb:dataset <#tdb_prod_dataset> ;
  tdb:graphName <http://data.hulib.helsinki.fi/attx/ids> ;
  .

<#attx-prov> rdf:type tdb:GraphTDB ;
  tdb:dataset <#tdb_prod_dataset> ;
  tdb:graphName <http://data.hulib.helsinki.fi/attx/prov> ;
  .

<#tdb_prod_dataset> rdf:type tdb:DatasetTDB ;
  tdb:location "DB" ;
  ## Query timeout on this dataset (milliseconds)
  ja:context [ ja:cxtName "arq:queryTimeout" ;  ja:cxtValue "3000" ] ;
  ## Default graph for query is the (read-only) union of all named graphs.
  tdb:unionDefaultGraph true ;
  .

# Data
<#attx-onto> ja:externalContent <file:data/attx-onto.ttl> .

```

### Querying the Default Graph

The Default Graph is the graph set to be the union of all other graphs and can be queried as follows:

```turtle
SELECT ?subject ?predicate ?object
WHERE { GRAPH <urn:x-arq:DefaultGraph> {
    ?subject ?predicate ?object
  }
}
```

### Adding Reasoning Capability

The reasoning capabilities can be added by the following lines for a specific dataset:
```turtle
<#dataset>
    rdf:type    ja:RDFDataset ;
    ja:defaultGraph <#model> ;
    .      

######
# Default Model : Inference rules (OWL, here)
<#model> a ja:InfModel;
    ja:baseModel <#tdbGraph>;
    ja:reasoner
    [ ja:reasonerURL
        <http://jena.hpl.hp.com/2003/OWLFBRuleReasoner>
    ]
    .
```

Reasoners available in Jena for Fuseki configuration:
* `http://jena.hpl.hp.com/2003/OWLFBRuleReasoner`
* `http://jena.hpl.hp.com/2003/RDFSExptRuleReasoner`
* `http://jena.hpl.hp.com/2003/TransitiveReasoner`
* `http://jena.hpl.hp.com/2003/GenericRuleReasoner`
* `http://jena.hpl.hp.com/2003/DAMLMicroReasonerFactory`
* `http://jena.hpl.hp.com/2003/OWLFBRuleReasoner`
* `http://jena.hpl.hp.com/2003/OWLMicroFBRuleReasoner`
* `http://jena.hpl.hp.com/2003/OWLMiniFBRuleReasoner`
* `http://jena.hpl.hp.com/2003/DIGReasoner`

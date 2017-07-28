# Overview

This document describes how to setup a local cache of Library of Congress (LoC) Names linked data for query and use in an end-user application. 

## locnames vocabulary

The Library of Congress (LoC) Names authority is a copy of the names authority available at for search interactively at http://id.loc.gov.  At this writing, LoC does not provide an API for searching their linked data.  The recommendation is to take a full download of the authority and create a search over a local cache.

## Background

The system described here is designed to process linked data search results into a format that is easily digestable by end applications.  It allows user interfaces to be designed to handle searching and presentation of linked data in a consistent way.  

There are a number of authorities that are configured to work with this process.  Look for all repositories starting with `qa_` in [Linked Data for Libraries Labs (LD4L Labs)](https://github.com/ld4l-labs).

There are two servers that drive this system.

1. linked data authority server - This server can be controlled by the authority provider or you can use a locally cached version of the authority.  It must be querable via curl with the query passed as a parameter.  Additionally, it must return a serialization of rdf (e.g. rdf/xml, jsonld, ntripled, turtle, etc.)

2. questioing authority normalization server - This server is an installing of the [questioning authority (qa) gem](https://rubygems.org/gems/qa).  A configuration file specific for NALT is installed which tells QA how to extract information from the linked data search results in order to normalize the results that are passed to the application.

More information on working with linked data through the QA gem is available in it's README in section [Linked Open Data (LOD) Authorities](https://github.com/samvera/questioning_authority#linked-open-data-lod-authorities)


# Usage

## Setting up a linked data authority server for LoC Names

### Use the external LoC Names authority server

This is not available for search at this time.  You can access a single term URI and get a serialization of linked data results back.

### Use a local cache of NALT authority data

You can create a local cache of LoC Names authority data by [downloading](http://id.loc.gov/download/) LC Name Authority File (MADS/RDF only) and putting it into a local triplestore.  There are a number of triplestores that you can use for this purpose.

#### Our implementation

* We stored linked data in a Jena/Fuseki triplestore with a lucene index supporting the query search.
* We split our data to support sub-authorities for personal_name, corporate_name, and title based on rdf:type.
* We added support for limiting the number of returned results (i.e. maxRecords param).
* We added support for filtering by language (i.e. lang param).
* We add a ranking predicate (i.e. <http://vivoweb.org/ontology/core#rank>) to search results to allow the results to be populated in a graph and maintain search results ranking. 

## Setting up a QA server to include the LoC Names authority

### Create a QA server app IF this is your first authority

Create a rails app and install the QA gem by following the instructions in the QA gem in section [How do I use this?](https://github.com/samvera/questioning_authority#how-do-i-use-this)

NOTE: Hyrax applications have QA built in.  You can follow the instructions for an existing QA server with the Hyrax app acting as the server.  You still might want to create a separate QA server app that lives on an optimized machine or to share the authority access between multiple apps.  See [Hyrax documentation](https://github.com/samvera/hyrax/wiki/Autocomplete-in-Hyrax) for more information.

### For existing and new QA servers

1. Add the configuration for LoC Names from [config/authorities/linked_data/locnames.json](https://github.com/ld4l-labs/qa_locnames/tree/master/config/authorities/linked_data/locnames.json) to the same directory location in your server app.
1. Edit locnames.json 
  1. update `search -> url -> template` and `term -> url -> template` to be the URL to the linked data authority server that you are using.  
  1. If your server returns linked data in the same predicate structures as LoC, then you shouldn't have to make extensive changes to the  configuration settings.  Minimally, you will need to update the `search -> url -> mapping` for the search URL parameters and `term -> url -> mapping` for the term URL parameters.  You will want to verify that your serialization includes all the predicates mentioned in the configration.  You may want to augment your triplestore to support the parameters for limiting the number of results and filtering languages.   

More information on writing QA linked data configurations is available in it's README in section [Configuring a LOD Authority](https://github.com/samvera/questioning_authority#configuring-a-lod-authority).

## Debugging

Because there are a number of systems integrating with each other, you should test the generated links at each level.

### Test that the linked data authority server returns expected RDF serialization.

For a cache triplestore server like Jena/Fuseki, this will be something like...

http://localhost:3005/ld4l_services/loc_name_batch.jsp?query=test&maxRecords=3&entity=PersonalName

```
<http://id.loc.gov/authorities/names/n82045653> <http://vivoweb.org/ontology/core#rank> "1" .
_:690d9c82b060eed1d7a81d3a3df29146 <http://id.loc.gov/ontologies/RecordInfo#languageOfCataloging> <http://id.loc.gov/vocabulary/iso639-2/eng> .
_:690d9c82b060eed1d7a81d3a3df29146 <http://id.loc.gov/ontologies/RecordInfo#recordChangeDate> "2014-05-16T07:03:20" .
_:690d9c82b060eed1d7a81d3a3df29146 <http://id.loc.gov/ontologies/RecordInfo#recordContentSource> <http://id.loc.gov/vocabulary/organizations/dlc> .
_:690d9c82b060eed1d7a81d3a3df29146 <http://id.loc.gov/ontologies/RecordInfo#recordStatus> "revised" .
_:690d9c82b060eed1d7a81d3a3df29146 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://id.loc.gov/ontologies/RecordInfo#RecordInfo> .
_:d7035f100a9c11b135e119a024167dab <http://id.loc.gov/ontologies/RecordInfo#languageOfCataloging> <http://id.loc.gov/vocabulary/iso639-2/eng> .
_:d7035f100a9c11b135e119a024167dab <http://id.loc.gov/ontologies/RecordInfo#recordChangeDate> "1982-04-05T00:00:00" .
_:d7035f100a9c11b135e119a024167dab <http://id.loc.gov/ontologies/RecordInfo#recordContentSource> <http://id.loc.gov/vocabulary/organizations/dlc> .
_:d7035f100a9c11b135e119a024167dab <http://id.loc.gov/ontologies/RecordInfo#recordStatus> "new" .
_:d7035f100a9c11b135e119a024167dab <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://id.loc.gov/ontologies/RecordInfo#RecordInfo> .
<http://id.loc.gov/authorities/names/n82045653> <http://purl.org/dc/terms/created> "1982-04-05T00:00:00" .
<http://id.loc.gov/authorities/names/n82045653> <http://purl.org/dc/terms/modified> "2014-05-16T07:03:20" .
<http://id.loc.gov/authorities/names/n82045653> <http://www.loc.gov/mads/rdf/v1#adminMetadata> _:690d9c82b060eed1d7a81d3a3df29146 .
<http://id.loc.gov/authorities/names/n82045653> <http://www.loc.gov/mads/rdf/v1#adminMetadata> _:d7035f100a9c11b135e119a024167dab .
<http://id.loc.gov/authorities/names/n82045653> <http://www.loc.gov/mads/rdf/v1#authoritativeLabel> "Twain, Mark, 1835-1910 (Spirit)"@en .
<http://id.loc.gov/authorities/names/n82045653> <http://www.loc.gov/mads/rdf/v1#hasExactExternalAuthority> <http://viaf.org/viaf/sourceID/LC%7Cn+82045653#skos:Concept> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.loc.gov/mads/rdf/v1#hasSource> _:10a20a2a812b63e6b4889e5a3d01087d .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.loc.gov/mads/rdf/v1#Authority> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.loc.gov/mads/rdf/v1#PersonalName> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2004/02/skos/core#Concept> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/2004/02/skos/core#exactMatch> <http://viaf.org/viaf/sourceID/LC%7Cn+82045653#skos:Concept> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/2004/02/skos/core#inScheme> <http://id.loc.gov/authorities/names> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/2004/02/skos/core#member> <http://id.loc.gov/authorities/names/collection_LCNAF> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/2004/02/skos/core#member> <http://id.loc.gov/authorities/names/collection_NamesAuthorizedHeadings> .
<http://id.loc.gov/authorities/names/n82045653> <http://www.w3.org/2004/02/skos/core#prefLabel> "Twain, Mark, 1835-1910 (Spirit)"@en .
```

### Test that QA returns the expected normalized data

For the attached configuration, with the server running on port 3002 on your local machine, this will be something like...

http://localhost:3002/qa/search/linked_data/locnames/personal_name?q=mark+twain&maxRecords=3

```
[
  {"uri":"http://id.loc.gov/authorities/names/n79021164",
   "id":"http://id.loc.gov/authorities/names/n79021164",
   "label":"Twain, Mark, 1835-1910"
  },
  {"uri":"http://id.loc.gov/authorities/names/n82045653",
   "id":"http://id.loc.gov/authorities/names/n82045653",
   "label":"Twain, Mark, 1835-1910 (Spirit)"
  },
  {"uri":"http://id.loc.gov/authorities/names/no2009013312",
   "id":"http://id.loc.gov/authorities/names/no2009013312",
   "label":"Twain, Norman"
  }
]
```

### Test that QA generates the expected linked data authority server URL

If you don't see any results, you can check that the correct URL for the linked data server is generated.  In the QA server, search log/development.log for `QA Linked Data search url:`.  Then you can copy/paste that URL into a browser or use curl in the terminal to verify that the generate URL accessing the linked data authority server actually returns data as expected.

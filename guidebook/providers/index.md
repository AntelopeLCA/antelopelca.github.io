---
layout: page
title: Guidebook -- Data Providers
permalink: /guidebook/providers
---

A "Provider" is a python class that understands how to access a certain kind of LCA data source.  Providers 
implement the `BasicArchive` (`antelope_core.archives.basic_archive.BasicArchive`) interface, which (though 
poorly specified) provides a set of useful functions for storing and retrieving entities.  

## The `EntityStore`

The `EntityStore` is essentially a big hash table that allows you to retrieve an "entity" from a reference string.  An entity store has a handful of properties, of which the most useful are:

 - `ref` - the semantic reference for the data collection (this becomes the `origin` of a data set)
 - `source` - a file, directory, or URL that contains the data collection
 - `static` - a boolean property that indicates (if true) that the entire data collection must be loaded at once.

Its core utilities are:
 - `retrieve_or_fetch_entity(entity_id)`, which loads an entity by its id, and stores it in local memory
 - `_add(entity, ref)`, which stores entities in a way that handles validation and retrieval
 - `__getitem__`, which retrieves an already-loaded entity by its reference or UUID
 - `load_all()`, which loads all content
 - `serialize()`, which writes the entity store to a JSON file.

It is a partially-abstract class, and the following routines must be implemented by providers:
 - `_load_all()` which is called by `load_all()`
 - `_fetch()` which is called by `retrieve_or_fetch_entity()` when the entity is not known locally

In addition, each provider is responsible for constructing valid entities.

> The provider infrastructure is some of the oldest python code in the archive. please don't judge.
{: .prompt-tip }


The `BasicArchive` implements the key features of the `EntityStore` for flows and quantities, as well as introduces a `search()` function. A `BasicArchive` can be saved and restored from JSON and forms the base class for all other providers.  The `LcArchive` adds processes to the list of supported entities, and is used for all Life Cycle data sources that contain processes.

## Default Providers

The providers available built-in in `antelope_core` are:
 - `BasicArchive` (generic container for quantities and flows)
 - `LcArchive` (adds processes)
 - `Background` (the "trivial" background engine, for accessing files containing rolled-up datasets)
 - `OpenLcaJsonLdArchive` (for OpenLCA .zip files)
 - `EcoinventLcia` (for ecoinvent-issued LCIA tables)
 - `Traci21Factors` (for accessing TRACI 2.1 spreadsheet)
 - `XdbClient` (for connecting to `xdb` background data servers)

Adding `lxml` support (`$ pip install lxml`) allows XML-based providers to be loaded:
 - `EcospoldV1Archive` for ecospold v1, including ecoinvent 2.2 and old-style US LCI
 - `EcospoldV2Archive` for ecospold v2, including ecoinvent 3.x databases
 - `IlcdArchive` for ILCD datasets (note: this has not been maintained for some time)
 - `IlcdLcia` is a subclass of `IlcdArchive` and adds the capability to read stored LCIA results 

Adding `antelope_background` introduces Tarjan ordering:
 - `TarjanBackground` performs partial ordering of databases and constructs allocated LCI matrices

Adding `antelope_foreground` introduces foreground modeling capacity:
 - `LcForeground` provides the capability to create and save fragments, and also stores catalog references to other data sources
 - `OryxClient` (for connecting to `oryx` foreground data servers)

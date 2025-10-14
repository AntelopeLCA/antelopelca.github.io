---
layout: page
title: Guidebook - QDB (Quantity Database)
permalink: /guidebook/qdb
---

The Quantity Database (QDB) is Antelope's system for managing LCIA (Life Cycle Impact Assessment) characterization factors, flowables, and environmental contexts. It provides a unified interface for querying and analyzing impact assessment data from multiple sources.

## Overview

The QDB aggregates LCIA methods, flowables, and contexts from various databases into a single queryable system. This allows you to:
- Search for characterization factors across multiple LCIA methods
- Explore flowable synonyms and alternative names
- Navigate context hierarchies (environmental compartments)
- Compare impact assessment methods
- Build custom LCIA analyses

## Key Concepts

### Flowables
Flowables represent flows of materials or energy in life cycle inventories. Examples include:
- Carbon dioxide (CO2)
- Electricity
- Water
- Nitrogen oxides (NOx)

The QDB tracks synonyms and alternative names for flowables across different databases.

### Contexts
Contexts represent environmental compartments or locations where flows occur. Examples include:
- emission/air
- emission/water/ground water
- resource/in ground

Contexts are hierarchical, with parent-child relationships (e.g., "air" contains "urban air" and "rural air").

### Quantities
Quantities in the QDB represent LCIA methods and impact categories. Examples include:
- TRACI 2.1: Global Warming
- ReCiPe 2016: Acidification
- IPCC 2013: Climate Change (GWP 100)

Each quantity has associated characterization factors that link flowables and contexts to impact values.

## Using QDB

### Web Interface

The [QDB Dash](/guidebook/qdb-dash) application provides a user-friendly web interface for exploring and analyzing the QDB. It allows you to:
- Search for flowables, contexts, and quantities
- View detailed information about each entity
- Build selections and run analyses
- Export results in multiple formats

See the [QDB Dash documentation](/guidebook/qdb-dash) for complete usage instructions.

### Python API

(Documentation coming soon - programmatic access to QDB through the Antelope catalog system)

## Data Sources

The QDB can integrate data from various sources:
- **LCIA method packages**: openLCA methods, TRACI, etc.
- **Flow lists**: Federal Commons flow list, ecoinvent flows
- **Custom databases**: Local or organization-specific LCIA data

Configuration of data sources is handled through the Antelope catalog system.

## Related Resources

- [QDB Dash Web Interface](/guidebook/qdb-dash) - Interactive web application
- [Antelope Quickstart](/guidebook/quickstart) - Getting started with Antelope
- [Glossary](/guidebook/glossary) - Definitions of key terms

[guidebook home](/guidebook)

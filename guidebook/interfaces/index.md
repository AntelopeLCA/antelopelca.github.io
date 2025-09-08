---
layout: page
title: Antelope Interface Reference
permalink: /guidebook/interfaces/
parent: /guidebook
---

Antelope organizes LCA functionality into seven distinct interfaces, each providing access to different types of operations and data. This modular design allows data sources to implement only the interfaces they support, while clients can discover available functionality dynamically.

## Origins and Queries

Data sources are organized semantically into hierarchically-specified "origins" that indicate the data
resource and applicable version. An origin is like a "domain name" for LCA data. For instance:

 * `ecoinvent.3.10.cutoff`
 * `lcacommons.uslci.fy24.q4`
 * `openlca.lcia`

Origins are hierarchically specified, and the first "dot" entry indicates the "issuer", the party that is ostensibly responsible for maintaining the data (in the above examples: `ecoinvent`, `lcacommons`, `openlca`). In current practice, Antelope maintainers are responsible for all data content.

Work in antelope is performed using a "catalog", which connects origins to data sources and allows a user to retrieve data by creating a "query". The objects returned by the query are (almost) always "references" to data entities. These references contain a captive query and are instrumented with all the different query methods appropriate to each entity type.

```python
from antelope_core import LcCatalog
catalog = LcCatalog()
catalog.new_resource('my.origin', '/path/to/data/source', 'DataSourceType', interfaces=('basic', 'exchange'))

catalog.index_ref('my.origin')
fs = catalog.query('my.origin').flows(name="polyethylene")
```

## The Seven Interfaces

LCA information is organized into several "interfaces" that (a) represent different forms of knowledge about the data and (b) can be implemented using different back-end systems.  

Under Antelope, reference data sources are all considered to be *read-only*, meaning that their contents may not be altered by a user.  All user modifications are intended to occur within the [foreground](foreground).

| Interface                    | Purpose                                        | Key Operations                        | Access Type    |
|------------------------------|------------------------------------------------|---------------------------------------|----------------|
| [**Basic**](basic)           | Entity retrieval and properties                | get(), properties(), validate()       | Read-only      |
| [**Index**](index-interface) | Search and enumeration                         | processes(), flows(), count()         | Read-only      |
| [**Exchange**](exchange)     | Process inventories and exchange relationships | inventory(), exchanges(), ev()        | Read-only      |
| [**Background**](background) | Matrix operations and system LCI               | lci(), sys_lci(), consumers()         | Read-only      |
| [**Quantity**](quantity)     | Unit conversion and LCIA                       | cf(), do_lcia(), factors()            | **Read-Write** |
| [**Configure**](configure)   | Resource configuration; allocation             | set_reference(), characterize_flow()  | **Read-Write** |
| [**Foreground**](foreground) | Foreground modeling and observation            | new_fragment(), observe(), traverse() | **Read-Write** |

## Interface Dependencies

The interfaces build upon each other in layers:

- **Basic**: Foundation for all other interfaces
- **Exchange + Index**: Required together for inventory modeling  
- **Background**: Requires Exchange + Index for matrix construction
- **Quantity**: Works with any of the above for LCIA operations
- **Configure**: Mainly oriented towards linking and allocation to prepare background LCI data
- **Foreground**: Uses all interfaces for comprehensive modeling


## Qualitative and quantitative information 

Each interface may include both qualitative (structured non-numeric) and quantitative (numeric) information.  In access-controlled settings, access grants include a boolean flag for `values` which indicates whether the grantee is authorized to receive numerical information.

Quantitative information varies by interface according to the following table:

| Interface  | Qualitative information                            | Quantitative Information       |
|------------|----------------------------------------------------|--------------------------------|
| Basic      | Entity documentation                               | Aggregated LCIA results        |
| Index      | Searchability                                      | Linking data                   |
| Exchange   | Exchanges                                          | Exchanges with exchange values |
| Background | Ordering (foreground, background, emission, cutoff | LCI Results                    | 
| Quantity   | Characterizations                                  | Characterization values        |

## Query Pattern

All interfaces follow the same query delegation pattern:

```python
# Get a query for a specific data source
query = catalog.query('my.data.source')

# Check what interfaces are available
print(query.origin, query.interfaces)

# Use interface methods directly
process = query.get('process-id')           # Basic interface
inventory = query.inventory(process)        # Exchange interface
results = query.do_lcia(quantity, inventory) # Quantity interface
```

Once an entity *reference* is retrieved via a query, it can be used to access applicable  methods directly.

```python
inv_query = catalog.query('inventory.resource')
process = query.get('process-id')
process.inventory()

lcia_query = catalog.query('lcia.resource')
lcia = lcia_query.get('lcia-method-id')

result = process.bg_lcia(lcia)
```

## Interface Discovery

Data sources advertise which interfaces they support:

```python
# Check available interfaces
catalog.show_interfaces()
# output: my.data.source [basic, exchange, quantity]

# Attempt operations only on supported interfaces
if 'index' in query.interfaces:
    processes = list(query.processes())
```

## Next Steps

- Browse individual interface documentation for detailed method references
- See the [Quick Start Guide](/guidebook/quickstart) for practical examples
- Review [Provider Development](/guidebook/providers) to implement interfaces

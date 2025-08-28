---
layout: page
title: Antelope Interface Reference
permalink: /guidebook/interfaces/
---

# Antelope Interface Reference

Antelope organizes LCA functionality into seven distinct interfaces, each providing access to different types of operations and data. This modular design allows data sources to implement only the interfaces they support, while clients can discover available functionality dynamically.

## The Seven Interfaces

| Interface | Purpose | Key Operations |
|-----------|---------|----------------|
| [**Basic**](basic/) | Entity retrieval and properties | get(), properties(), validate() |
| [**Exchange**](exchange/) | Process inventories and exchange relationships | inventory(), exchanges(), ev() |
| [**Index**](index-interface/) | Search and enumeration | processes(), flows(), count() |
| [**Quantity**](quantity/) | Unit conversion and LCIA | cf(), do_lcia(), factors() |
| [**Background**](background/) | Matrix operations and system LCI | lci(), sys_lci(), check_bg() |
| [**Configure**](configure/) | System configuration and tuning | set_reference(), characterize_flow() |
| [**Foreground**](foreground/) | Fragment modeling and observation | new_fragment(), observe(), traverse() |

## Interface Dependencies

The interfaces build upon each other in layers:

- **Basic**: Foundation for all other interfaces
- **Exchange + Index**: Required together for inventory modeling  
- **Quantity**: Works with any of the above for LCIA operations
- **Background**: Requires Exchange + Index for matrix construction
- **Configure**: Enhances any data source with configuration capabilities
- **Foreground**: Uses all interfaces for comprehensive modeling

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
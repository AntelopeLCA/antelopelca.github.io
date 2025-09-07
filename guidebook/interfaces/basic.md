---
layout: page
title: Basic Interface
permalink: /guidebook/interfaces/basic/
---

# Basic Interface

The Basic interface provides fundamental entity retrieval and documentary information access. Because different interfaces can be implemented independently, it is possible to envision a resource without basic interface access; however, the information provided in the basic interface is fundamental to understanding and human interpretation of the data available in other interfaces.

## Identity Definition and Retrieval

The Basic interface treats data sources as collections of **entities** (processes, flows, quantities) with **properties** (metadata attributes).  An entity is uniquely specified by its *origin* and its *external reference* (`external_ref`). The `external_ref` is often, though not always, a UUID.  The `origin` and `external_ref`, concatenated by `/`, form a "link", which provides enough information to retrieve the entity:

```text
link = 'lcacommons.uslci.fy24.q1.01/0aaf1e13-5d80-37f9-b7bb-81a6b8965c71'
origin, external_ref = link.split('/')
catalog.query(origin).get(external_ref).show()
'''
ProcessRef catalog reference (0aaf1e13-5d80-37f9-b7bb-81a6b8965c71)
origin: lcacommons.uslci.fy24.q1.01
   UUID: 0aaf1e13-5d80-37f9-b7bb-81a6b8965c71
   Name: Petroleum refining, at refinery
Comment: 
==Local Fields==
                  @type: Process
        Classifications: ['31-33: Manufacturing', '3241: Petroleum and Coal Products Manufacturing']
           SpatialScope: United States
          TemporalScope: {'begin': '2009-01-01', 'end': '2023-12-31'}
defaultAllocationMethod: PHYSICAL_ALLOCATION
            description: This gate-to-gate unit process is for net production of 1 kilogram of general refinery product as well as data allocated to specific refinery...
'''
```

For entities whose `external_ref` is not a UUID, it is typical (but not required) for the entity to *also* have a canonical UUID.

## Key Methods

### Entity Retrieval

#### `get(entity_id, **kwargs)`
Retrieve an entity by its identifier.
```python
process = query.get('72d5a381-8cae-4e1d-b0a3-26cc43b69867')
flow = query.get('electricity-mix-us')
```

#### `get_uuid(external_ref)` 
Get the canonical UUID for an entity, in case it is different from the `external_ref`.
```python
uuid = query.get_uuid('process-name')
```

### Property Access
All entity properties are *case-insensitive*. 

#### `properties(external_ref, **kwargs)`
List all available properties for an entity.
```python
props = query.properties('process-id')
# Returns: ['Name', 'Comment', 'SpatialScope', 'TemporalScope', ...]
```

#### `get_item(external_ref, property_name)`
Access a specific property value.
```python
name = query.get_item('process-id', 'Name')
location = query.get_item('process-id', 'SpatialScope')
```

#### Access from Entity
Given an EntityRef object, properties can be accessed directly:
```python
p_ref = query.get('entity_id')
list(p_ref.properties())
# ['Name', 'Comment', 'SpatialScope', 'TemporalScope', ...]
p_ref.get_item('Name')
# returns the process's name
p_ref['Name'] 
```

#### Setting an Entity's Local Properties
An EntityRef's properties can be set within a session, but those property settings are not propagated back
to the data source (which is immutable)
# returns the process's name
p_ref['Name'] = 'Temporary name'  # sets the name of the *local* EntityRef
```

### System Operations

#### `validate()`
Verify that the query is connected to a valid, operational data source.
```python
if query.validate():
    print("Data source is accessible")
else:
    print("Data source unavailable")
```

#### `synonyms(term, **kwargs)`
Find alternative terms for flowables, quantities, or contexts.
```python
terms = query.synonyms('carbon dioxide')
# Returns: ['CO2', 'carbon dioxide', 'CO₂']
```

### Context Management

#### `get_context(term, **kwargs)`
Retrieve standardized context information for environmental compartments.
```python
context = query.get_context('air')
context = query.get_context('water, groundwater')
```

#### `is_lcia_engine(**kwargs)`
Determine if the data source uses standardized contexts and flowables.
```python
if query.is_lcia_engine():
    # Uses canonical terms across data sources
    print("Standardized terminology")
else:
    # Uses native/provincial terms
    print("Source-specific terminology")
```

### Advanced Operations

#### `get_reference(external_ref)`
Get the reference information for an entity (typically reference exchanges for processes).
```python
refs = query.get_reference('process-id')
```

#### `bg_lcia(process, quantity=None, ref_flow=None, **kwargs)`
Retrieve pre-computed background LCIA results (if available and authorized).
```python
result = query.bg_lcia(process, quantity='GWP 100')
```

## Usage Patterns

### Basic Entity Inspection
```python
# Get entity and examine its properties
entity = query.get('some-id')
print(f"Entity: {entity}")

# List all properties
for prop in query.properties('some-id'):
    value = query.get_item('some-id', prop)
    print(f"{prop}: {value}")
```

### Data Source Validation
```python
def check_data_source(query):
    if not query.validate():
        raise ConnectionError(f"Cannot access {query.origin}")
    
    print(f"Connected to {query.origin}")
    if query.is_lcia_engine():
        print("Uses standardized terminology")
```

## Error Handling

The Basic interface defines standard exceptions:

- **`EntityNotFound`**: Requested entity doesn't exist
- **`ItemNotFound`**: Requested property doesn't exist  
- **`NoAccessToEntity`**: Entity exists but access is denied
- **`ValidationError`**: Data source validation failed
- **`BasicRequired`**: Operation requires Basic interface (none found)

## Implementation Notes

### Required for All Queries
Every functional Antelope query must implement the Basic interface. It's the minimum requirement for data source connectivity.

### Property Access Pattern
The `get_item()` method is the primary way to access entity metadata. Properties are implementation-dependent but common ones include:
- `Name`: Human-readable name
- `Comment`: Description or notes
- `SpatialScope`: Geographic applicability  
- `TemporalScope`: Time period validity
- `Classifications`: Category assignments

### Reference vs. Entity
The Basic interface works with both "entities" (full objects) and "entity references" (lightweight proxies). References are returned by catalog queries and can be dereferenced using Basic interface methods.

## Next Steps

- Learn about [Exchange Interface](../exchange/) for inventory operations
- See [Index Interface](../index/) for search capabilities
- Review [Quick Start Guide](/guidebook/quickstart) for practical examples

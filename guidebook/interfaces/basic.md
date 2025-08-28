---
layout: page
title: Basic Interface
permalink: /guidebook/interfaces/basic/
---

# Basic Interface

The Basic interface provides fundamental entity retrieval and documentary information access. It serves as the foundation for all other Antelope interfaces and is required for any valid query.

## Core Concept

The Basic interface treats data sources as collections of **entities** (processes, flows, quantities) with **properties** (metadata attributes). It provides the essential "get entity by ID" and "get property by name" operations that underpin all LCA data access.

## Key Methods

### Entity Retrieval

#### `get(entity_id, **kwargs)`
Retrieve an entity by its identifier.
```python
process = query.get('72d5a381-8cae-4e1d-b0a3-26cc43b69867')
flow = query.get('electricity-mix-us')
```

#### `get_uuid(external_ref)` 
Get the canonical UUID for an entity.
```python
uuid = query.get_uuid('process-name')
```

### Property Access

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
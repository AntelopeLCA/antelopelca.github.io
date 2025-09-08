---
layout: page
title: Index Interface  
permalink: /guidebook/interfaces/index-interface/
---

# Index Interface

The Index interface provides search, enumeration, and discovery capabilities for LCA data sources. It enables users to explore data contents, find entities by criteria, and understand relationships between flows and processes.

## Core Concept

The Index interface treats data sources as searchable catalogs of entities. It answers questions like:
- "How many processes are in this database?"
- "What processes produce electricity?"
- "Which flows contain the term 'carbon'?"
- "What LCIA methods are available?"

Unlike the Basic interface which requires knowing specific entity IDs, the Index interface enables **discovery** of entities through search and enumeration.

## Key Methods

### Entity Counting and Enumeration

#### `count(entity_type, **kwargs)`
Count entities of a specific type.
```python
n_processes = query.count('process')
n_flows = query.count('flow')  
n_quantities = query.count('quantity')
print(f"Database contains {n_processes} processes, {n_flows} flows")
```

#### `processes(**kwargs)`, `flows(**kwargs)`, `quantities(**kwargs)`
Generate entities with keyword filtering.
```python
# All processes
for process in query.processes():
    print(f"Process: {process}")

# Keyword search
electricity_processes = list(query.processes(Name='electricity'))
steel_flows = list(query.flows(Name='steel'))
```

### Flow and Context Discovery

#### `flowables(search=None, **kwargs)`
Generate known flowable names (substance/material identifiers).
```python
# All flowables
all_flowables = list(query.flowables())

# Search for carbon-containing substances  
carbon_flowables = list(query.flowables(search='carbon'))
# Returns: ['carbon dioxide', 'carbon monoxide', 'carbon black', ...]
```

#### `contexts(**kwargs)`
Generate environmental contexts (compartments).
```python
contexts = list(query.contexts())
# Returns: [('air',), ('water', 'surface water'), ('soil',), ...]
```

### LCIA Method Discovery

#### `lcia_methods(**kwargs)`
Generate LCIA methods (impact categories).
```python
methods = list(query.lcia_methods())
for method in methods:
    print(f"Method: {method}")
    # Example: "TRACI 2.1 - Climate Change - kg CO2-eq"
```

#### `lcia(**kwargs)`
Generate LCIA-related quantities (broader than methods).
```python
lcia_quantities = list(query.lcia())
```

### Reference Exchanges

#### `targets(flow, direction=None, **kwargs)`
Find processes that include the specified flow as a reference exchange.  Optionally, specify a direction (`Input` or `Output`).

```python
# Who can consume electricity?
electricity_consumers = list(query.targets(electricity_flow, direction='Input'))

# Who can provide steel?  
steel_producers = list(query.targets(steel_flow, direction='Output'))
```

### Flow Matching and Validation

#### `unmatched_flows(flows, **kwargs)`
Identify flows from a list that don't match known flowables.
```python
candidate_flows = ['carbon dioxide', 'CO2', 'unknown_substance', 'methane']
unmatched = list(query.unmatched_flows(candidate_flows))
# Returns: ['unknown_substance'] (if not in database)
```

## Usage Patterns

### Database Exploration
```python
def explore_database(query):
    """Get overview of database contents"""
    print(f"Database: {query.origin}")
    print(f"Processes: {query.count('process')}")
    print(f"Flows: {query.count('flow')}")  
    print(f"Quantities: {query.count('quantity')}")
    
    # Sample some processes
    processes = list(query.processes())[:5]
    print("Sample processes:")
    for p in processes:
        print(f"  {p}")
```

### Flow Network Analysis
```python
fs = list(query.flows(name='steel'))
producers = list(fs[0].targets())
if len(producers) == 1:
    print("Single producer (unique source)")
elif len(producers) == 0:
    print("No producers (cutoff flow)")
else:
    print(f"Multiple producers (choice required)")
```

## Error Handling

- **`IndexRequired`**: Operation requires Index interface but none found
- **`InvalidDirection`**: Invalid direction specification provided
- **`InvalidSense`**: Invalid sense specification (Source/Sink vs Input/Output)

## Implementation Notes

### Index Creation Required
Most data sources require explicit indexing before Index interface operations work:
```python
# Create index for a data source
catalog.index_ref('my.data.source')

# Now index operations work
processes = list(query.processes())
```

### Generator Pattern
Most Index methods return generators, not lists:
```python
# Efficient for large datasets
for process in query.processes():
    if some_condition(process):
        break
        
# Convert to list when needed
process_list = list(query.processes())
```

## Next Steps

- Learn about [Quantity Interface](../quantity/) for LCIA operations
- See [Background Interface](../background/) for system analysis
- Review [Exchange Interface](../exchange/) for inventory operations

---
layout: page
title: Background Interface
permalink: /guidebook/interfaces/background/
---

# Background Interface

The Background interface provides system-level LCA operations through matrix-based calculations. It performs Life Cycle Inventory (LCI) computations by solving technology matrices, enabling full cradle-to-gate impact assessment.

## Core Concept

The Background interface treats LCA databases as systems of linear equations represented by matrices:
- **Technology Matrix (A)**: Process-to-process dependencies  
- **Intervention Matrix (B)**: Process-to-environment emissions
- **Demand Vector (y)**: Products requested from the system

It solves **Ax = y** to find process activity levels, then calculates **Bx** for environmental interventions.

This enables **system LCI**: tracing through supply chains to calculate complete environmental profiles, rather than just direct process inventories.

## Key Methods

### System Setup and Validation

#### `check_bg(reset=False, **kwargs)`
Initialize or validate the background system.
```python
# Create/verify background matrices
query.check_bg()

# Force recreation with new parameters
query.check_bg(reset=True, multi_term='cutoff')
```

### Process Analysis

#### `lci(process, ref_flow=None, **kwargs)`
Get complete system LCI for a process (cradle-to-gate).
```python
# Complete supply chain inventory
lci_result = query.lci(steel_process)
for exchange in lci_result:
    print(f"{exchange.direction}: {exchange.value} {exchange.flow}")
```

#### `sys_lci(demand, **kwargs)`  
Perform LCI on an arbitrary demand vector.
```python
# Custom demand specification
from antelope import UnallocatedExchange
demand = [
    UnallocatedExchange(steel_process, steel_flow, 'Output', value=1000),
    UnallocatedExchange(aluminum_process, aluminum_flow, 'Output', value=500)
]

lci_result = query.sys_lci(demand)
```

### Flow Network Analysis

#### `consumers(process, ref_flow=None, **kwargs)`
Find processes that use the given process as input.
```python
# Who consumes steel?
steel_consumers = list(query.consumers(steel_process))
for consumer in steel_consumers:
    print(f"Consumer: {consumer.process}")
```

#### `emitters(flow, direction=None, **kwargs)`  
Find processes that emit a specific flow.
```python
# Who emits CO2?
co2_emitters = list(query.emitters(co2_flow, direction='Output'))
```

### Exchange Classification

#### `dependencies(process, ref_flow=None, **kwargs)`
Get intermediate (process-to-process) exchanges.
```python
# Supply chain dependencies
deps = list(query.dependencies(steel_process))
for dep in deps:
    print(f"Depends on: {dep.termination}")
```

#### `emissions(process, ref_flow=None, **kwargs)`
Get elementary (process-to-environment) exchanges.
```python  
# Environmental emissions
emissions = list(query.emissions(steel_process))
for emission in emissions:
    print(f"Emits: {emission.flow} to {emission.termination}")
```

#### `cutoffs(process, ref_flow=None, **kwargs)`
Get unlinked (cutoff) exchanges.
```python
# Unlinked flows
cutoffs = list(query.cutoffs(steel_process))
```

### System Structure Analysis

#### `foreground_flows(search=None, **kwargs)`
List processes in the foreground (user-controllable) portion.
```python
foreground = list(query.foreground_flows())
print(f"Foreground contains {len(foreground)} processes")
```

#### `background_flows(search=None, **kwargs)`  
List processes in the background (system) portion.
```python
background = list(query.background_flows()) 
print(f"Background contains {len(background)} processes")
```

#### `exterior_flows(direction=None, search=None, **kwargs)`
List flows that cross the system boundary.
```python
# System inputs/outputs
exterior = list(query.exterior_flows())
for flow in exterior:
    print(f"System {flow.direction}: {flow.flow}")
```

### Advanced Matrix Operations

#### `foreground(process, ref_flow=None, **kwargs)`
Get foreground matrix entries for a process.
```python
# Foreground dependencies (controllable)
fg_exchanges = list(query.foreground(steel_process))
```

#### `ad(process, ref_flow=None, **kwargs)`
Get aggregated dependencies on background system.
```python
# Dependencies on background processes
bg_deps = list(query.ad(steel_process))
```

#### `bf(process, ref_flow=None, **kwargs)`
Get aggregated environmental interventions.
```python
# Aggregated emissions from full supply chain
interventions = list(query.bf(steel_process))
```

### System-Level LCIA

#### `sys_lcia(process, query_qty, observed=None, ref_flow=None, **kwargs)`
Perform system-level LCIA with supply chain impacts.
```python
# Complete climate impact including supply chain
climate_impact = query.sys_lcia(steel_process, gwp_quantity)
print(f"Total GWP: {climate_impact.total()} kg CO2-eq")
```

#### `deep_lcia(process, quantity_ref, ref_flow=None, **kwargs)`
Perform LCIA at the technology matrix level.
```python
# Matrix-level impact assessment
deep_result = query.deep_lcia(steel_process, gwp_quantity)
```

## Usage Patterns

### Complete LCA Workflow
```python
def perform_complete_lca(query, process, impact_methods):
    """Perform comprehensive LCA with system boundaries"""
    
    print(f"Analyzing {process} with system boundaries...")
    
    # Get system LCI
    lci = query.lci(process)
    print(f"System LCI includes {len(list(lci))} flows")
    
    # Calculate impacts
    results = {}
    for method_name, quantity in impact_methods.items():
        impact = query.sys_lcia(process, quantity)
        results[method_name] = impact.total()
        print(f"{method_name}: {impact.total()}")
    
    return results
```

### Supply Chain Analysis  
```python
def analyze_supply_chain(query, target_process):
    """Analyze supply chain structure"""
    
    # Direct dependencies
    deps = list(query.dependencies(target_process))
    print(f"Direct suppliers: {len(deps)}")
    
    # System boundaries
    fg_flows = list(query.foreground_flows())
    bg_flows = list(query.background_flows())
    
    print(f"Foreground processes: {len(fg_flows)}")
    print(f"Background processes: {len(bg_flows)}")
    
    # Check if target is in background system
    is_bg = query.is_in_background(target_process)
    print(f"{target_process} in background: {is_bg}")
```

### System Validation
```python  
def validate_system_linking(query):
    """Check system completeness and linking"""
    
    try:
        # Test system initialization
        query.check_bg()
        print("Background system initialized successfully")
        
        # Check for cutoffs
        exterior = list(query.exterior_flows())
        cutoffs = [f for f in exterior if f.termination is None]
        
        print(f"Total exterior flows: {len(exterior)}")
        print(f"Cutoff flows: {len(cutoffs)}")
        
        if cutoffs:
            print("Warning: System has cutoffs")
            for cutoff in cutoffs[:5]:  # Show first 5
                print(f"  {cutoff.flow}")
                
    except Exception as e:
        print(f"System validation failed: {e}")
```

## Error Handling

- **`BackgroundRequired`**: Operation requires Background interface but none found
- **`LinkingError`**: System linking failed (broken references, circular dependencies)  
- May also raise matrix computation errors from underlying libraries

## Implementation Notes

### Matrix Requirements
The Background interface requires complete, linkable data:
- All process dependencies must be terminable to other processes
- Technology matrix must be square and invertible
- Circular dependencies are resolved using Tarjan ordering

### Foreground vs Background  
The interface distinguishes between:
- **Foreground**: User-controllable processes (can be modified/substituted)
- **Background**: System processes (treated as fixed infrastructure)

### Authorization Levels
Some operations require specific permissions:
- Basic system queries may work with read-only access
- Matrix operations may require computational authorization
- System modification requires write access

### Performance Considerations
Background operations can be computationally intensive:
- Matrix factorization may take significant time for large systems
- Results are often cached for performance
- Use `check_bg()` to pre-initialize before heavy operations

## Integration Requirements

The Background interface typically requires:
- **Index interface**: For system structure analysis
- **Exchange interface**: For matrix construction  
- **Quantity interface**: For impact assessment operations

## Relationship to Other Interfaces

- **Exchange**: Provides inventory data for matrix construction
- **Index**: Used for process/flow discovery and linking validation
- **Quantity**: Enables system-level LCIA calculations
- **Configure**: Provides linking strategy configuration

## Next Steps

- Learn about [Configure Interface](../configure/) for system tuning
- See [Foreground Interface](../foreground/) for user-controlled modeling
- Review [Exchange Interface](../exchange/) for inventory foundations
---
layout: page
title: Exchange Interface
permalink: /guidebook/interfaces/exchange/
---

# Exchange Interface

The Exchange interface provides access to process inventories and the quantitative relationships between flows in LCA processes. It implements the **Exchange Relation**: given a process, reference flow, and query flow, it reports the quantity of the query flow exchanged per unit of reference flow.

## Core Concept

The Exchange interface models processes as collections of **exchanges** - quantified flows of materials, energy, or services. Each exchange connects a **parent process** to a **flow** with a **direction** (input/output) and **magnitude** (exchange value).

This interface answers questions like:
- "What inputs does this process consume?"
- "How much CO₂ is emitted per kWh of electricity produced?"
- "What is the full inventory of this process?"

## Key Methods

### Exchange Listing

#### `exchanges(process, **kwargs)`
Retrieve a process's complete exchange list without quantitative values.
```python
exchanges = list(query.exchanges(process))
for ex in exchanges:
    print(f"{ex.direction}: {ex.flow} -> {ex.termination}")
```

### Quantitative Exchange Relations

#### `ev(process, flow, direction=None, termination=None, ref_flow=None, **kwargs)`
Get the exchange value - the fundamental quantitative relationship.
```python
# CO2 emissions per unit of process activity
co2_rate = query.ev(process, co2_flow, direction='Output')

# Energy input per kg of product output  
energy_per_kg = query.ev(process, energy_flow, 
                        direction='Input', ref_flow=product_flow)
```

#### `exchange_values(process, flow, direction=None, termination=None, reference=None, **kwargs)`
Retrieve detailed exchange information with values (somewhat deprecated).
```python
exchanges = query.exchange_values(process, electricity_flow, direction='Input')
```

### Process Inventories

#### `inventory(process, ref_flow=None, scenario=None, **kwargs)`
Get the complete inventory of a process, optionally normalized to a reference flow.

**Unallocated inventory** (no reference flow):
```python
# All exchanges including reference flows
inventory = query.inventory(process)
for ex in inventory:
    print(f"{ex.direction}: {ex.value} {ex.flow}")
```

**Allocated inventory** (with reference flow):
```python
# Non-reference exchanges per unit of reference
inventory = query.inventory(process, ref_flow=main_product)
for ex in inventory:
    print(f"{ex.direction}: {ex.value} {ex.flow} per {ref_flow}")
```

### LCIA Integration

#### `contrib_lcia(process, quantity=None, ref_flow=None, **kwargs)`
Perform contribution analysis of a process's LCIA impact.
```python
# Impact contribution by exchange
contributions = query.contrib_lcia(process, quantity=gwp_quantity)
for contrib in contributions:
    print(f"{contrib.flow}: {contrib.cumulative_result} kg CO2-eq")
```

### Advanced Relations

#### `exchange_relation(process, ref_flow, exch_flow, direction, termination=None, **kwargs)`
Get detailed exchange relation information (analogous to `quantity.quantity_relation`).
```python
relation = query.exchange_relation(process, product_flow, input_flow, 'Input')
```

## Usage Patterns

### Process Inventory Analysis
```python
def analyze_process(query, process):
    print(f"Analyzing {process}")
    
    # Get reference flows
    refs = list(query.references(process))
    main_ref = refs[0] if refs else None
    
    if main_ref:
        # Get allocated inventory
        inventory = query.inventory(process, ref_flow=main_ref.flow)
        
        inputs = [ex for ex in inventory if ex.direction == 'Input']
        outputs = [ex for ex in inventory if ex.direction == 'Output']
        
        print(f"Inputs per {main_ref.flow}:")
        for inp in inputs:
            print(f"  {inp.value} {inp.flow}")
            
        print(f"Outputs per {main_ref.flow}:")
        for out in outputs:
            print(f"  {out.value} {out.flow}")
```

### Exchange Value Lookup
```python
def get_specific_exchange(query, process, target_flow, direction):
    """Find exchange value for specific flow and direction"""
    try:
        value = query.ev(process, target_flow, direction=direction)
        return value
    except ExchangeRequired:
        print(f"No {direction.lower()} of {target_flow} found in {process}")
        return None
```

### Multi-Product Process Handling
```python
def handle_multiproduct_process(query, process):
    """Handle processes with multiple reference flows"""
    inventory = query.inventory(process)  # unallocated
    
    refs = [ex for ex in inventory if ex.is_reference]
    deps = [ex for ex in inventory if not ex.is_reference]
    
    print(f"Reference products: {len(refs)}")
    for ref in refs:
        print(f"  {ref.value} {ref.flow}")
        
    print(f"Shared inputs/outputs: {len(deps)}")
```

## Error Handling

- **`ExchangeRequired`**: Operation requires Exchange interface but none found
- May also raise Basic interface exceptions for entity access

## Implementation Notes

### Reference Flow Behavior
The behavior of `inventory()` depends critically on the reference flow parameter:

1. **No reference flow**: Returns all exchanges including references (unallocated)
2. **Valid reference flow**: Returns non-reference exchanges normalized to reference unit
3. **Invalid reference flow**: Behavior is implementation-dependent

### Allocation Handling
When a process has multiple reference flows (co-products), the Exchange interface handles allocation:
- Some implementations require explicit allocation configuration
- Others may apply default allocation rules
- The `configure` interface provides allocation control

### Fragment Integration
For fragment entities (from foreground modeling), the Exchange interface:
- Accepts `scenario` parameter instead of `ref_flow`
- May traverse fragment trees to compute inventories
- Integrates with foreground modeling workflows

### Values Permission
Some Exchange interface operations require `values=True` authorization:
- Quantitative methods like `ev()` and `inventory()`
- Documentary methods like `exchanges()` may work without values

## Relationship to Other Interfaces

- **Basic**: Required for entity access and properties
- **Index**: Often used together for process discovery
- **Quantity**: Provides LCIA analysis of exchange inventories  
- **Background**: Uses Exchange data to build technology matrices
- **Configure**: Provides allocation and reference flow management

## Next Steps

- Learn about [Index Interface](../index/) for process discovery
- See [Quantity Interface](../quantity/) for LCIA operations
- Review [Background Interface](../background/) for system-level analysis
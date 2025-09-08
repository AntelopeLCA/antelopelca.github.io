---
layout: page
title: Exchange Interface
permalink: /guidebook/interfaces/exchange/
parent: /guidebook/interfaces/
---

# Exchange Interface

The Exchange interface provides access to process inventories and the quantitative relationships between flows in LCA processes. It implements the **Exchange Relation**: given a process, reference flow, and query flow, it reports the quantity of the query flow exchanged per unit of reference flow.

## Core Concept

The Exchange interface models processes as collections of **exchanges** - quantified flows of materials, energy, or services. Each exchange connects a **parent process** to a **flow** with a **direction** (input/output) and **magnitude** (exchange value).

This interface answers questions like:
- "What inputs does this process consume?"
- "How much CO₂ is (directly) emitted per kWh of electricity produced?"
- "What is the full inventory of this process?"

## Key Methods

### Exchange Listing

#### `exchanges(process, **kwargs)`
Retrieve a process's complete exchange list without quantitative values.

Returns a generator of [`ExchangeRef`](/guidebook-exchanges#exchangerefs) objects. Note that a `termination` for a linked exchange is an `external_ref` for a process, not the target process itself.
```python
process = query.get('my_process_id')
for ex in sorted(process.exchanges())
    if ex.type == 'node':
        tgt = query.get(ex.termination)
        print(f"{ex.direction}: {ex.flow} -> {tgt.name}")
    elif ex.type == 'context':
        print(f"{ex.direction}: {ex.flow} -> {ex.termination.name}")
    else:
        print(f"{ex.direction}: {ex.flow}  ({ex.type})")
```

### Quantitative Exchange Relations


#### `exchange_values(process, flow, direction=None, termination=None, reference=None, **kwargs)`
Retrieve detailed exchange information with values (somewhat deprecated).
```python
exchanges = query.exchange_values(process, electricity_flow, direction='Input')
```

#### `ev(process, flow, direction=None, termination=None, ref_flow=None, **kwargs)`
Get the exchange value per unit of reference flow - the fundamental quantitative relationship.  Returns a float. If the process has multiple references, one must be specified.
```python
# CO2 emissions per unit of process activity
process = query.get('process ref')
rx = process.reference()  # a reference exchange
co2_flows = list(query.flows(name='carbon dioxide'))
co2_flow = co2_flows[7]
energy_flows = list(k for k in process.exchanges() if k.flow.unit == 'MJ')

co2_per_kg = process.ev(co2_flow, direction='Output')

total_mj_per_kg = sum(process.ev(process, energy_flow, 
                                 direction='Input', ref_flow=rx.flow)
                      for energy_flow in energy_flows)
```
### Process Inventories

#### `inventory(process, ref_flow=None, scenario=None, **kwargs)`
Get the complete inventory of a process, optionally normalized to a reference flow.

**Unallocated inventory** (no reference flow):
```python
# All exchanges including reference flows
inventory = query.inventory('process ref')
for i, ex in enumerate(inventory):
    print('%3d %s' % (i, ex))
```

**Allocated inventory** (with reference flow):
```python
# Non-reference exchanges per unit of reference
inventory = process.inventory(ref_flow=main_product)
print(process.reference(main_product))
for ex in inventory:
    print(ex)
```

### LCIA Integration

#### `contrib_lcia(process, quantity=None, ref_flow=None, **kwargs)`
Perform contribution analysis of a process's LCIA impact.
```python
# Impact contribution by exchange
result = query.contrib_lcia(process, quantity=gwp_quantity)
result.show_components()
```

### Advanced Relations

#### `exchange_relation(process, ref_flow, exch_flow, direction, termination=None, **kwargs)`
Get detailed exchange relation information (analogous to `quantity.quantity_relation`).  This answers the
question, "how much of `exch_flow` is (input/output) per unit of `ref_flow`?"  If 
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

- **`ExchangeRequired`**: Operation requires Exchange interface but none found / not authorized
- May also raise Basic interface exceptions for entity access

## Implementation Notes

### Reference Flow Behavior
The behavior of `inventory()` depends critically on the reference flow parameter:

1. **No reference flow**: Returns all exchanges including references (unallocated)
2. **Valid reference flow**: Returns non-reference exchanges normalized to reference unit
3. **Invalid reference flow**: Behavior is implementation-dependent

### Values Permission
Some Exchange interface operations require `values=True` authorization:
- Quantitative methods like `ev()` and `inventory()`
- Documentary methods like `exchanges()` return value-free results

## Relationship to Other Interfaces

- **Basic**: Required for entity access and properties
- **Index**: Often used together for process discovery
- **Quantity**: Provides LCIA analysis of exchange inventories  
- **Background**: Uses Exchange data to build LCI matrices
- **Configure**: Used to specify allocation and reference flow linking in preparation for building LCI matrices. 

## Next Steps

- Learn about [Index Interface](../index/) for process discovery
- See [Quantity Interface](../quantity/) for LCIA operations
- Review [Background Interface](../background/) for system-level analysis

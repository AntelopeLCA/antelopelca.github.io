---
layout: page
title: Quantity Interface
permalink: /guidebook/interfaces/quantity/
---

# Quantity Interface

The Quantity interface handles unit conversion and Life Cycle Impact Assessment (LCIA). It implements the **Quantity Relation**: mapping flows and their properties to impact categories through characterization factors.

## Core Concept

The Quantity interface models the relationships between:
- **Flows** (materials, energy, emissions)
- **Quantities** (units of measurement, impact categories) 
- **Characterization factors** (conversion coefficients)

It answers questions like:
- "How many kg CO₂-equivalent does 1 kg methane contribute to climate change?"
- "What is the total carbon footprint of this process inventory?"
- "What characterization factors are available for this flow?"

## Key Methods

### Characterization Factors

#### `cf(flow, quantity, ref_quantity=None, context=None, locale='GLO', strategy=None, **kwargs)`
Get a characterization factor - the core quantity conversion operation.
```python
# Climate change impact of methane emissions
cf_methane = query.cf(methane_flow, gwp_quantity)
# Returns: 25.0 (kg CO2-eq per kg methane)

# Energy content of diesel fuel  
energy_cf = query.cf(diesel_flow, energy_quantity)
# Returns: 43.0 (MJ per kg diesel)
```

#### `factors(quantity, flowable=None, context=None, **kwargs)`
List all characterization factors for an impact category.
```python
# All climate change factors
gwp_factors = list(query.factors(gwp_quantity))
for factor in gwp_factors:
    print(f"{factor.flowable}: {factor.value} {factor.unit}")
```

### Flow Characterization Profiles

#### `profile(flow, **kwargs)`
Get all characterizations available for a specific flow.
```python
# What impacts can be calculated for CO2?
co2_profile = list(query.profile(co2_flow))
for char in co2_profile:
    print(f"{char.quantity}: {char.value}")
    # Climate Change: 1.0 kg CO2-eq
    # Acidification: 0.0 kg SO2-eq  
    # etc.
```

### LCIA Calculations

#### `do_lcia(quantity, inventory, locale='GLO', **kwargs)`
Perform LCIA on a process inventory or exchange list.
```python
# Get process inventory
inventory = query.inventory(steel_process)

# Calculate climate impact
climate_result = query.do_lcia(gwp_quantity, inventory)
print(f"Total GWP: {climate_result.total()} kg CO2-eq")

# Show contribution details
for detail in climate_result.details():
    print(f"{detail.flow}: {detail.result} kg CO2-eq")
```

#### `lcia(process, ref_flow, quantity_ref, **kwargs)`
Perform complete process LCIA (legacy method, may be deprecated).
```python
result = query.lcia(process, ref_flow, gwp_quantity)
```

### Advanced Quantity Relations

#### `quantity_relation(flowable, ref_quantity, query_quantity, context, locale='GLO', **kwargs)`
Get detailed quantity relationship information.
```python
# Detailed conversion information
relation = query.quantity_relation(
    flowable='methane',
    ref_quantity=mass_quantity, 
    query_quantity=gwp_quantity,
    context=('air',),
    locale='GLO'
)
print(f"Conversion: {relation.value}")
print(f"Locale: {relation.locale}")
print(f"Origin: {relation.origin}")
```

### Data Management

#### `characterize(flowable, ref_quantity, query_quantity, value, context=None, location='GLO', **kwargs)`
Add new characterization data (typically used internally).
```python
# Add characterization factor
query.characterize(
    flowable='new_chemical',
    ref_quantity=mass_quantity,
    query_quantity=gwp_quantity, 
    value=150.0,
    context=('air',),
    location='GLO'
)
```

#### `get_canonical(quantity, **kwargs)`
Get the canonical/standard version of a quantity.
```python
# Standardize quantity reference
canonical_gwp = query.get_canonical('climate change')
canonical_gwp = query.get_canonical('GWP 100')
canonical_gwp = query.get_canonical('kg CO2 eq')
# All return the same canonical GWP quantity
```

### Normalization

#### `norm(quantity_ref, region=None, **kwargs)`
Get normalization factors for impact categories.
```python
# EU normalization factor for climate change
norm_factor = query.norm(gwp_quantity, region='Europe')
normalized_result = total_impact / norm_factor
```

## Usage Patterns

### Impact Assessment Workflow
```python
def assess_process_impacts(query, process, impact_methods):
    """Calculate multiple impact categories for a process"""
    
    # Get process inventory
    inventory = query.inventory(process)
    
    results = {}
    for method_name, quantity in impact_methods.items():
        try:
            result = query.do_lcia(quantity, inventory)
            results[method_name] = result.total()
            print(f"{method_name}: {result.total()} {quantity.unit}")
        except Exception as e:
            print(f"Could not calculate {method_name}: {e}")
    
    return results

# Usage
impact_methods = {
    'Climate Change': gwp_quantity,
    'Acidification': acidification_quantity,
    'Eutrophication': eutrophication_quantity
}
results = assess_process_impacts(query, steel_process, impact_methods)
```

### Characterization Factor Discovery
```python
def explore_characterization_coverage(query, quantity):
    """See what flows can be characterized for an impact method"""
    
    factors = list(query.factors(quantity))
    print(f"Impact method: {quantity}")
    print(f"Characterized flows: {len(factors)}")
    
    # Group by context
    by_context = {}
    for factor in factors:
        context = factor.context or 'Unspecified'
        if context not in by_context:
            by_context[context] = []
        by_context[context].append(factor)
    
    for context, context_factors in by_context.items():
        print(f"\n{context}: {len(context_factors)} flows")
        for factor in sorted(context_factors, key=lambda x: x.value, reverse=True)[:5]:
            print(f"  {factor.flowable}: {factor.value}")
```

### Unit Conversion
```python
def convert_units(query, flow, value, from_quantity, to_quantity):
    """Convert between different units for the same flow"""
    
    # Get conversion factor
    cf = query.cf(flow, to_quantity, ref_quantity=from_quantity)
    
    if cf:
        converted_value = value * cf
        print(f"{value} {from_quantity.unit} = {converted_value} {to_quantity.unit}")
        return converted_value
    else:
        print(f"No conversion available from {from_quantity} to {to_quantity}")
        return None

# Example: Convert energy units
convert_units(query, natural_gas_flow, 1000, volume_quantity, energy_quantity)
# Output: 1000 m³ = 35000 MJ
```

## Error Handling

- **`QuantityRequired`**: Operation requires Quantity interface but none found
- **`NoFactorsFound`**: No characterization factors available for the requested conversion
- **`ConversionReferenceMismatch`**: Reference quantities don't match for conversion
- **`FlowableMismatch`**: Flow specification doesn't match available characterizations

## Implementation Notes

### Term Management
The Quantity interface behavior depends on the underlying **term manager**:

- **Provincial mode** (`is_lcia_engine() = False`): Uses exact flowable/context names from the data source
- **LCIA Engine mode** (`is_lcia_engine() = True`): Uses standardized, canonical terms across data sources

### Locale Handling
Many methods accept a `locale` parameter (default 'GLO') for geographic-specific factors:
```python
# Use region-specific factors when available
us_cf = query.cf(flow, quantity, locale='US')
european_cf = query.cf(flow, quantity, locale='EU')
```

### Strategy Parameters
Some methods accept `strategy` parameters for handling multiple matches:
- `'first'`: Use first available factor
- `'last'`: Use last available factor  
- `'cutoff'`: Treat as cutoff if ambiguous
- `'abort'`: Raise error if ambiguous

### Result Objects
LCIA operations return rich result objects with:
- `total()`: Overall impact score
- `details()`: Contribution breakdown by flow
- `components()`: Grouping by component/process
- Serialization and display methods

## Relationship to Other Interfaces

- **Basic**: Required for entity access and properties
- **Exchange**: Provides inventories for LCIA calculations
- **Index**: Used to discover available LCIA methods
- **Configure**: Allows adding new characterization factors
- **Background**: Enables system-level impact assessment

## Next Steps

- Learn about [Background Interface](../background/) for system-level LCI/LCIA
- See [Exchange Interface](../exchange/) for inventory operations  
- Review [Configure Interface](../configure/) for adding characterizations
---
layout: page
title: Configure Interface
permalink: /guidebook/interfaces/configure/
---

# Configure Interface

The Configure interface provides tools for tuning and customizing LCA data sources. It allows users to modify reference flows, add characterization factors, configure allocation methods, and establish context mappings.

## Core Concept

The Configure interface enables **post-processing** of LCA data to:
- Fix data quality issues
- Customize process behaviors  
- Add missing characterization factors
- Establish terminological mappings
- Configure allocation strategies

Unlike other interfaces that access existing data, Configure **modifies** the data source to improve its utility and accuracy.

## Key Methods

### Reference Flow Management

#### `set_reference(process_ref, flow_ref, direction=None, **kwargs)`
Designate an exchange as a reference flow for a process.
```python
# Make electricity output the reference for power plant
query.set_reference(power_plant, electricity_flow, direction='Output')
```

#### `unset_reference(process_ref, flow_ref, direction=None, **kwargs)`
Remove reference status from an exchange.
```python
# Remove reference designation
query.unset_reference(power_plant, steam_flow, direction='Output')
```

### Allocation Configuration

#### `allocate_by_quantity(process_ref, quantity_ref, **kwargs)`
Configure multi-output process allocation using a specific quantity.
```python
# Allocate refinery outputs by economic value
query.allocate_by_quantity(refinery_process, economic_value_quantity)

# Allocate by mass
query.allocate_by_quantity(steel_process, mass_quantity)
```

### Characterization Management

#### `characterize_flow(flow_ref, quantity_ref, value, location='GLO', **kwargs)`
Add characterization factors to flows.
```python
# Add climate impact for new chemical
query.characterize_flow(
    new_chemical_flow,
    gwp_quantity, 
    value=120.0,  # kg CO2-eq per kg
    location='GLO'
)

# Add regional-specific factor
query.characterize_flow(
    methane_flow,
    gwp_quantity,
    value=28.0,
    location='IPCC AR5'
)
```

### Context and Terminology Mapping

#### `context_hint(local_term, canonical_context, **kwargs)`
Map source-specific context names to canonical contexts.
```python
# Map local context names to standard contexts
query.context_hint('air emissions', ('air',))
query.context_hint('water discharge', ('water', 'surface water'))
query.context_hint('waste', ('technosphere',))
```

### Configuration Validation

#### `check_config(config, c_args, **kwargs)`
Validate that a configuration option is valid.
```python
# Check if allocation configuration is valid
is_valid = query.check_config('allocate_by_quantity', (process, quantity))
```

## Usage Patterns

### Data Quality Improvement
```python
def fix_reference_flows(query, process_list):
    """Fix common reference flow issues"""
    
    for process in process_list:
        # Get current references
        refs = list(process.references())
        
        if len(refs) == 0:
            print(f"Warning: {process} has no reference flows")
            # Could set one based on heuristics
            
        elif len(refs) > 1:
            print(f"Info: {process} has {len(refs)} reference flows")
            # Multi-output process - may need allocation
            
        # Check for problematic references
        for ref in refs:
            if ref.direction == 'Input':
                print(f"Unusual: {process} has input reference {ref.flow}")
```

### Characterization Factor Addition
```python
def add_missing_characterizations(query, flow_list, impact_method):
    """Add characterization factors for flows missing them"""
    
    for flow in flow_list:
        try:
            # Check if characterization exists
            cf = query.cf(flow, impact_method)
            print(f"{flow}: {cf} (existing)")
            
        except NoFactorsFound:
            # Prompt user or use default value
            print(f"{flow}: No characterization found")
            
            # Example: Add default factor
            if 'CO2' in flow.name:
                query.characterize_flow(flow, impact_method, value=1.0)
                print(f"Added default CO2 factor for {flow}")
```

### Multi-Output Process Configuration
```python
def configure_multioutput_processes(query):
    """Set up allocation for multi-output processes"""
    
    # Find processes with multiple outputs
    processes = query.processes()
    
    for process in processes:
        refs = list(process.references(direction='Output'))
        
        if len(refs) > 1:
            print(f"Multi-output process: {process}")
            print(f"Outputs: {[ref.flow for ref in refs]}")
            
            # Configure allocation method
            # Option 1: Economic allocation
            try:
                economic_qty = query.get_canonical('economic value')
                query.allocate_by_quantity(process, economic_qty)
                print(f"Configured economic allocation for {process}")
            except:
                # Option 2: Mass allocation
                try:
                    mass_qty = query.get_canonical('mass')
                    query.allocate_by_quantity(process, mass_qty)
                    print(f"Configured mass allocation for {process}")
                except:
                    print(f"Could not configure allocation for {process}")
```

### Context Standardization
```python
def standardize_contexts(query, origin):
    """Map local context names to canonical ones"""
    
    # Common context mappings
    context_mappings = {
        'air emissions': ('air',),
        'atmospheric emissions': ('air',),  
        'water emissions': ('water',),
        'water discharge': ('water', 'surface water'),
        'wastewater': ('water', 'ground water'),
        'soil emissions': ('soil',),
        'ground': ('soil',),
        'waste': ('technosphere',)
    }
    
    for local_term, canonical in context_mappings.items():
        try:
            query.context_hint(local_term, canonical)
            print(f"Mapped '{local_term}' -> {canonical}")
        except Exception as e:
            print(f"Could not map '{local_term}': {e}")
```

## Error Handling

- **`ConfigRequired`**: Operation requires Configure interface but none found
- **`NotImplementedError`**: Configuration option not supported by implementation
- May also raise validation errors for invalid configuration parameters

## Implementation Notes

### Persistence
Configuration changes may be:
- **Temporary**: Applied only for current session
- **Persistent**: Saved to data source or configuration files
- **Cached**: Stored in intermediate representations

Implementation behavior varies - check documentation for specific providers.

### Validation
The Configure interface often validates changes:
- Reference flows must be existing exchanges
- Allocation quantities must be characterized for all reference flows
- Context mappings must reference valid contexts

### Side Effects
Configuration changes can have broad impacts:
- Changing reference flows affects inventory calculations
- Adding characterizations enables new LCIA calculations
- Allocation changes modify all process inventories

## Authorization Requirements

Configure operations often require elevated permissions:
- **Read-write access** to data sources
- **Configuration authority** for system-wide changes
- **Validation permissions** for data quality modifications

## Configuration Strategies

### Allocation Methods
Common allocation approaches:
```python
# Mass allocation (physical)
query.allocate_by_quantity(process, mass_quantity)

# Economic allocation (value-based)  
query.allocate_by_quantity(process, price_quantity)

# Energy allocation (thermodynamic)
query.allocate_by_quantity(process, energy_quantity)
```

### Reference Flow Selection
Guidelines for reference flow configuration:
- Choose **main product** for single-output processes
- Consider **economic significance** for multi-output processes
- Ensure **data quality** for chosen references
- Maintain **consistency** across similar processes

## Relationship to Other Interfaces

- **Basic**: Provides entity access for configuration
- **Exchange**: Modified by reference flow and allocation changes
- **Quantity**: Enhanced by characterization factor additions
- **Background**: Benefits from improved linking through configuration
- **Index**: May require re-indexing after configuration changes

## Next Steps

- Learn about [Foreground Interface](../foreground/) for user modeling
- See [Background Interface](../background/) for system-level operations
- Review [Quantity Interface](../quantity/) for characterization methods
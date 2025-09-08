---
layout: page
title: Foreground Interface
permalink: /guidebook/interfaces/foreground/
---

# Foreground Interface

The Foreground interface enables construction and management of user-defined LCA models using **fragments**. It provides tools for creating modular, reusable product system models that can incorporate data from multiple sources.

## Core Concept

The Foreground interface implements **fragment-based modeling**, where:
- **Fragments** represent observed material flows between activities
- **Fragment trees** model complete product systems as hierarchical structures  
- **Observations** capture quantitative and qualitative model parameters
- **Scenarios** enable exploration of different model configurations

This approach enables **data-free modeling**: models can be shared and reused while preserving confidentiality of specific data values.

## Key Methods

### Entity Creation

#### `new_quantity(name, ref_unit=None, **kwargs)`
Create custom quantities for modeling.
```python
# Create custom impact category
custom_impact = query.new_quantity(
    'Water consumption impact',
    ref_unit='m³ water-eq'
)
```

#### `new_flow(name, ref_quantity=None, context=None, **kwargs)`
Create custom flows for product system modeling.
```python
# Create product flow
widget_flow = query.new_flow(
    'Custom widget',
    ref_quantity=mass_quantity
)

# Create elementary flow
waste_flow = query.new_flow(
    'Industrial waste',
    context=('technosphere',)
)
```

### Fragment Construction

#### `new_fragment(flow, direction, **kwargs)`
Create fragment nodes representing observed exchanges.
```python
# Create reference fragment (product output)
widget_fragment = query.new_fragment(
    widget_flow, 
    direction='Output'
)

# Create child fragment (material input)
steel_input = query.new_fragment(
    steel_flow,
    direction='Input', 
    parent=widget_fragment
)
```

#### `split_subfragment(fragment, replacement=None, **kwargs)`
Split fragments into reusable submodels.
```python
# Split complex fragment into reusable component
packaging_fragment = query.split_subfragment(
    complex_fragment,
    replacement=packaging_process
)
```

### Fragment Discovery and Navigation

#### `fragments_with_flow(flow, direction=None, **kwargs)`
Find fragments that use specific flows.
```python
# Find all fragments using electricity
electricity_fragments = list(query.fragments_with_flow(electricity_flow))

# Find fragments outputting steel
steel_outputs = list(query.fragments_with_flow(
    steel_flow, 
    direction='Output'
))
```

#### `child_flows(fragment, **kwargs)`
Get immediate children of a fragment.
```python
# Get direct inputs/outputs
children = list(query.child_flows(widget_fragment))
for child in children:
    print(f"{child.direction}: {child.flow}")
```

#### `tree(fragment, **kwargs)`
Get complete fragment tree structure.
```python
# Get full hierarchy
tree = query.tree(widget_fragment)
for node in tree:
    print(f"{'  ' * node.depth}{node.fragment}")
```

### Observation and Parameterization

#### `observe(fragment, exchange_value=None, anchor=None, name=None, scenario=None, **kwargs)`
Record quantitative observations and model parameters.
```python
# Observe exchange value (material input rate)
query.observe(
    steel_input,
    exchange_value=2.5,  # kg steel per widget
    name='Steel input rate'
)

# Observe termination (supply source)
query.observe(
    steel_input,
    anchor=steel_production_process,
    name='Steel supplier'
)

# Scenario-specific observation
query.observe(
    energy_input,
    exchange_value=15.0,
    scenario='renewable_energy'
)
```

#### `observe_unit_score(fragment, quantity, score, scenario=None, **kwargs)`
Record pre-computed impact scores.
```python
# Cache impact assessment result
query.observe_unit_score(
    steel_fragment,
    gwp_quantity,
    score=2.8,  # kg CO2-eq per kg steel
    scenario='low_carbon'
)
```

### Fragment Analysis

#### `traverse(fragment, scenario=None, **kwargs)`
Compute fragment activity levels and exchange values.
```python
# Traverse default scenario
traversal = query.traverse(widget_fragment)
for flow in traversal:
    print(f"{flow.fragment}: {flow.magnitude} {flow.flow}")

# Traverse specific scenario
renewable_traversal = query.traverse(
    widget_fragment,
    scenario='renewable_energy'
)
```

#### `activity(fragment, scenario=None, **kwargs)`
Get direct activity (first-level children only).
```python
# Direct material/energy inputs
activities = query.activity(widget_fragment)
for activity in activities:
    print(f"Direct: {activity.flow} = {activity.magnitude}")
```

#### `cutoff_flows(fragment, scenario=None, **kwargs)`
Identify unlinked flows requiring background data.
```python
# Find flows needing external data
cutoffs = query.cutoff_flows(widget_fragment)
for cutoff in cutoffs:
    print(f"Cutoff: {cutoff.flow} ({cutoff.direction})")
```

### Impact Assessment

#### `fragment_lcia(fragment, quantity_ref, scenario=None, **kwargs)`
Perform LCIA on fragment model.
```python
# Calculate climate impact
climate_result = query.fragment_lcia(
    widget_fragment,
    gwp_quantity,
    scenario='baseline'
)

print(f"Total impact: {climate_result.total()} kg CO2-eq")

# Show contributions
for component in climate_result.components():
    print(f"{component.fragment}: {component.result}")
```

### Model Management

#### `save(**kwargs)`
Persist foreground model to storage.
```python
# Save model and cached results
query.save(save_unit_scores=True)
```

#### `parameters(fragment=None, **kwargs)`
List observable parameters in the model.
```python
# All model parameters
all_params = list(query.parameters())

# Parameters for specific fragment
widget_params = list(query.parameters(fragment=widget_fragment))
```

#### `anchors(fragment, **kwargs)`
List termination anchors for a fragment.
```python
# Available supply options
anchors = list(query.anchors(steel_input))
for anchor in anchors:
    print(f"Option: {anchor}")
```

## Usage Patterns

### Product System Modeling
```python
def build_product_model(query, product_name, main_flow):
    """Build hierarchical product system model"""
    
    # Create reference fragment
    product = query.new_fragment(main_flow, 'Output')
    query.observe(product, name=product_name)
    
    # Add material inputs
    materials = [
        ('Steel', steel_flow, 2.5),
        ('Plastic', plastic_flow, 0.8),
        ('Electricity', electricity_flow, 12.0)
    ]
    
    for name, flow, amount in materials:
        material_input = query.new_fragment(
            flow, 'Input', parent=product
        )
        query.observe(
            material_input,
            exchange_value=amount,
            name=f"{name} input"
        )
    
    return product
```

### Scenario Analysis
```python
def compare_scenarios(query, fragment, scenarios):
    """Compare different model scenarios"""
    
    results = {}
    
    for scenario_name in scenarios:
        # Traverse scenario
        traversal = query.traverse(fragment, scenario=scenario_name)
        
        # Calculate impact
        impact = query.fragment_lcia(
            fragment,
            gwp_quantity, 
            scenario=scenario_name
        )
        
        results[scenario_name] = {
            'flows': len(list(traversal)),
            'impact': impact.total()
        }
        
        print(f"{scenario_name}: {impact.total()} kg CO2-eq")
    
    return results
```

### Model Validation and QC
```python
def validate_fragment_model(query, fragment):
    """Validate fragment model completeness"""
    
    print(f"Validating model: {fragment}")
    
    # Check for unobserved fragments
    tree = query.tree(fragment)
    unobserved = []
    
    for node in tree:
        if not node.observed:
            unobserved.append(node)
    
    if unobserved:
        print(f"Warning: {len(unobserved)} unobserved fragments")
        for node in unobserved:
            print(f"  {node.fragment}")
    
    # Check for cutoffs
    cutoffs = query.cutoff_flows(fragment)
    if cutoffs:
        print(f"Info: {len(cutoffs)} cutoff flows")
        for cutoff in cutoffs[:5]:  # Show first 5
            print(f"  {cutoff.flow}")
    
    # Check traversability
    try:
        traversal = list(query.traverse(fragment))
        print(f"Model traverses successfully ({len(traversal)} flows)")
    except Exception as e:
        print(f"Error: Model traversal failed: {e}")
```

### Modular Model Construction
```python
def create_modular_system(query):
    """Build system from reusable modules"""
    
    # Create reusable packaging module
    packaging = query.new_fragment(packaging_flow, 'Output')
    
    cardboard = query.new_fragment(
        cardboard_flow, 'Input', parent=packaging
    )
    query.observe(cardboard, exchange_value=0.1, name='Cardboard')
    
    # Create main product using packaging module
    product = query.new_fragment(widget_flow, 'Output')
    
    # Reference packaging module
    packaging_input = query.new_fragment(
        packaging_flow, 'Input', parent=product
    )
    query.observe(packaging_input, anchor=packaging, name='Packaging')
    
    return product, packaging
```

## Error Handling

- **`ForegroundRequired`**: Operation requires Foreground interface but none found
- May raise validation errors for malformed fragment structures
- Traversal errors for incomplete or circular fragment models

## Implementation Notes

### Fragment Types
- **Reference fragments**: Top-level product outputs
- **Child fragments**: Material/energy inputs and co-product outputs
- **Balance fragments**: Automatically computed flows (mass/energy balance)

### Observation Persistence
Observations can be:
- **Default**: Applied to all scenarios
- **Scenario-specific**: Applied only to named scenarios
- **Temporary**: Not persisted to storage

### Model Reusability
Fragment models support:
- **Substitution**: Replace anchors with different processes
- **Scaling**: Adjust exchange values proportionally  
- **Composition**: Embed fragments within larger models

## Authorization Requirements

Foreground operations typically require:
- **Write access** for model creation and modification
- **Save permissions** for model persistence
- **Access to background data** for anchor resolution

## Relationship to Other Interfaces

- **Basic**: Entity creation and property access
- **Exchange**: Background process integration via anchors
- **Quantity**: Impact assessment of fragment models
- **Index**: Background data discovery for anchors
- **Background**: System-level inventory calculations

## Next Steps

- Learn about [Background Interface](../background/) for system integration
- See [Quantity Interface](../quantity/) for impact assessment

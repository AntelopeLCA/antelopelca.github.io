---
layout: page
title: Fragment Discovery and Navigation
permalink: /guidebook/foreground/discovery/
---

# Fragment Discovery and Navigation

The advanced foreground implementation provides powerful tools for discovering and navigating fragment models. These capabilities enable users to work efficiently with complex fragment hierarchies and find specific model components.

## Fragment Search Methods

### Basic Fragment Listing

#### `fragments(show_all=False, **kwargs)`
List reference fragments (top-level models).
```python
# Get all reference fragments (product models)
models = list(query.fragments())
for model in models:
    print(f"Model: {model.external_ref}")

# Show ALL fragments (including children) - use with caution
all_frags = list(query.fragments(show_all=True))
```

### Smart Fragment Finding

#### `frag(string, many=False, **kwargs)`
Find fragments by UUID prefix or name.
```python
# Find fragment by UUID prefix
fragment = query.frag('a1b2c3d4')  # matches UUID starting with a1b2c3d4

# Find by exact name
steel_model = query.frag('steel_production_model')

# Handle multiple matches
try:
    fragment = query.frag('steel')  # Raises error if multiple matches
except ValueError as e:
    print(f"Multiple matches found: {e}")
    
# Get all matches instead
fragments = query.frag('steel', many=True)
```

#### `frags(string, **kwargs)`
Find named fragments by name prefix.
```python
# Find all fragments whose names start with 'steel'
steel_fragments = list(query.frags('steel'))

# Find energy-related models
energy_models = list(query.frags('energy'))
```

### Flow-Based Discovery

#### `fragments_with_flow(flow, direction=None, reference=True, background=None, **kwargs)`
Find fragments that use specific flows.
```python
# Find all fragments that output steel
steel_producers = list(query.fragments_with_flow(
    steel_flow, 
    direction='Output'
))

# Find fragments that consume electricity (inputs)
electricity_consumers = list(query.fragments_with_flow(
    electricity_flow,
    direction='Input'
))

# Find only reference fragments (product models)
steel_models = list(query.fragments_with_flow(
    steel_flow,
    direction='Output',
    reference=True
))

# Find only child fragments (components)
steel_inputs = list(query.fragments_with_flow(
    steel_flow,
    direction='Input', 
    reference=False
))
```

## Hierarchy Navigation

### Parent-Child Relationships

#### `parent(fragment, **kwargs)`
Get the parent of a fragment.
```python
# Navigate up the hierarchy
child = query.get('child-fragment-id')
parent = query.parent(child)
print(f"Parent: {parent.external_ref}")
```

#### `top(fragment, **kwargs)`
Get the top-level (reference) fragment.
```python
# Find the root of any fragment
deep_child = query.get('nested-fragment-id')
root = query.top(deep_child)
print(f"Top-level model: {root.external_ref}")
```

### Tree Structure Analysis

#### `tree(fragment, **kwargs)`
Get complete fragment tree structure.
```python
# Explore full model hierarchy
model = query.frag('widget_production')
tree = query.tree(model)

for node in tree:
    indent = '  ' * node.depth
    print(f"{indent}{node.fragment.external_ref}")
    print(f"{indent}  Flow: {node.fragment.flow}")
    print(f"{indent}  Direction: {node.fragment.direction}")
```

## Model Parameter Discovery

### Parameter Knobs

#### `knobs(search=None, **kwargs)`
Find "knobs" - named parameters that can be adjusted for scenario analysis.
```python
# Find all adjustable parameters
params = list(query.knobs())
for param in params:
    print(f"Parameter: {param.external_ref}")
    print(f"  Flow: {param.flow}")
    print(f"  Current value: {param.exchange_value()}")

# Search for specific parameters
energy_params = list(query.knobs(search='energy'))
```

### Scenario Discovery

#### `scenarios(fragment, recurse=True, **kwargs)`
Discover available scenarios in fragment models.
```python
# Find scenarios for a specific fragment
model = query.frag('steel_production')
scenarios = list(query.scenarios(model))
print(f"Available scenarios: {scenarios}")

# Find scenarios in entire model tree
all_scenarios = list(query.scenarios(model, recurse=True))
```

### Node Analysis

#### `nodes(origin=None, scenario=None, **kwargs)`
Find model nodes (termination anchors) for analysis.
```python
# Find all model terminations
nodes = list(query.nodes())
for node in nodes:
    print(f"Node: {node.fragment.external_ref}")
    print(f"  Terminates to: {node.termination}")
    print(f"  Magnitude: {node.magnitude}")

# Filter by data source origin
ecoinvent_nodes = list(query.nodes(origin='ecoinvent.3.8'))

# Filter by scenario
renewable_nodes = list(query.nodes(scenario='renewable_energy'))
```

## Usage Patterns

### Model Exploration Workflow
```python
def explore_model(query, model_name):
    """Comprehensive model exploration"""
    
    # Find the model
    model = query.frag(model_name)
    print(f"Exploring model: {model.external_ref}")
    
    # Show hierarchy
    print("\nModel structure:")
    tree = query.tree(model)
    for node in tree:
        indent = '  ' * node.depth
        print(f"{indent}{node.fragment.external_ref}")
    
    # Find adjustable parameters
    print("\nAdjustable parameters:")
    params = list(query.knobs())
    model_params = [p for p in params if query.top(p) == model]
    for param in model_params:
        print(f"  {param.external_ref}: {param.exchange_value()}")
    
    # Check scenarios
    scenarios = list(query.scenarios(model))
    if scenarios:
        print(f"\nAvailable scenarios: {scenarios}")
    
    return model
```

### Supply Chain Analysis
```python
def analyze_supply_chain(query, product_flow):
    """Analyze supply chain for a specific product"""
    
    # Find all producers of the product
    producers = list(query.fragments_with_flow(
        product_flow,
        direction='Output',
        reference=True
    ))
    
    print(f"Found {len(producers)} producers of {product_flow}")
    
    for producer in producers:
        print(f"\nProducer: {producer.external_ref}")
        
        # Find immediate inputs
        tree = query.tree(producer)
        inputs = [n for n in tree if n.fragment.direction == 'Input' 
                 and n.depth == 1]  # Direct children only
        
        print("Direct inputs:")
        for inp in inputs:
            print(f"  {inp.fragment.flow}: {inp.fragment.exchange_value()}")
```

### Cross-Model Analysis
```python
def find_shared_components(query, flows):
    """Find models that share common input flows"""
    
    # Find fragments for each flow
    shared_usage = {}
    
    for flow in flows:
        consumers = list(query.fragments_with_flow(flow, direction='Input'))
        
        for consumer in consumers:
            root = query.top(consumer)
            if root not in shared_usage:
                shared_usage[root] = []
            shared_usage[root].append(flow)
    
    # Show models with multiple shared flows
    print("Models with shared components:")
    for model, used_flows in shared_usage.items():
        if len(used_flows) > 1:
            print(f"{model.external_ref}: {[f.name for f in used_flows]}")
```

### Parameter Sensitivity Preparation
```python
def prepare_sensitivity_analysis(query, model):
    """Identify parameters for sensitivity analysis"""
    
    # Find all knobs in the model
    all_knobs = list(query.knobs())
    model_knobs = [k for k in all_knobs if query.top(k) == model]
    
    # Group by parameter type
    material_inputs = []
    energy_inputs = []
    other_params = []
    
    for knob in model_knobs:
        flow_name = knob.flow.name.lower()
        if 'energy' in flow_name or 'electricity' in flow_name:
            energy_inputs.append(knob)
        elif any(material in flow_name for material in ['steel', 'aluminum', 'plastic']):
            material_inputs.append(knob)
        else:
            other_params.append(knob)
    
    print("Parameters for sensitivity analysis:")
    print(f"Material inputs: {len(material_inputs)}")
    print(f"Energy inputs: {len(energy_inputs)}")  
    print(f"Other parameters: {len(other_params)}")
    
    return {
        'material': material_inputs,
        'energy': energy_inputs,
        'other': other_params
    }
```

## Best Practices

### Efficient Discovery
- Use `frag()` with specific prefixes rather than listing all fragments
- Filter `fragments_with_flow()` results by direction and reference status
- Cache discovery results when doing repeated analysis

### Navigation Patterns
- Always use `top()` to find the root model when working with child fragments
- Use `tree()` for comprehensive model structure understanding
- Use `parent()` for step-by-step navigation

### Parameter Management
- Use `knobs()` to find all adjustable model parameters
- Group parameters by type (material, energy, etc.) for organized analysis
- Use `scenarios()` to understand available model variants

## Next Steps

- Learn [Process Models](../process-models/) for background data integration
- Master [Advanced Operations](../advanced-operations/) for model manipulation
- Explore [Scenarios](../scenarios/) for multi-scenario analysis
---
layout: page
title: Process Model Integration
permalink: /guidebook/foreground/process-models/
---

# Process Model Integration

The advanced foreground implementation seamlessly converts background process data into foreground fragment models. This enables users to build detailed, modular models that combine proprietary data with public LCA databases.

## Core Process Integration Methods

### Creating Process Models

#### `create_process_model(process, ref_flow=None, StageName=None, **kwargs)`
Convert a background process into a fragment model.
```python
# Create fragment model from ecoinvent process
steel_process = catalog.get('ecoinvent.3.8', 'steel production')
steel_model = query.create_process_model(
    steel_process,
    StageName='Steel Production'
)

# Specify reference flow for multi-output processes
refinery_process = catalog.get('ecoinvent.3.8', 'petroleum refinery')
gasoline_model = query.create_process_model(
    refinery_process,
    ref_flow=gasoline_flow,
    StageName='Gasoline Production'
)
```

**What this creates:**
- **Reference fragment**: Product output with correct direction and exchange value
- **Balance fragment**: Internal node terminated to the original process
- **Metadata preservation**: StageName and other process properties

### Extending Process Models

#### `extend_process(fragment, scenario=None, include_elementary=False, inventory=False, **kwargs)`
Build out supply chain dependencies as child fragments.
```python
# Basic extension with intermediate flows
steel_model = query.create_process_model(steel_process)
query.extend_process(steel_model)

# Include elementary flows as fragments
query.extend_process(
    steel_model,
    include_elementary=True
)

# Use process inventory instead of background routes
query.extend_process(
    steel_model,
    inventory=True  # Preserves exchange metadata but may affect LCI accuracy
)

# Scenario-specific extension
query.extend_process(
    steel_model,
    scenario='renewable_energy'
)
```

**Extension behavior:**
- Creates child fragments for each process dependency
- Automatically terminates child fragments to background processes
- Preserves exchange values and metadata
- Respects existing child fragments (updates rather than duplicates)

### Fragment Creation from Exchanges

#### `fragment_from_exchanges(exchanges, parent=None, include_elementary=False, auto_anchor=True, **kwargs)`
Create or update fragments from exchange lists.
```python
# Create new model from exchange list
inventory = steel_process.inventory()
steel_model = query.fragment_from_exchanges(
    inventory,
    ref='steel_production_model',
    include_elementary=True
)

# Add children to existing fragment
aluminum_model = query.frag('aluminum_production')
aluminum_inputs = aluminum_process.dependencies()
query.fragment_from_exchanges(
    aluminum_inputs,
    parent=aluminum_model,
    auto_anchor=True  # Automatically link to existing fragments
)

# Update existing model with new data
updated_inventory = updated_steel_process.inventory()
query.fragment_from_exchanges(
    updated_inventory,
    parent=steel_model  # Updates existing children
)
```

## Advanced Integration Patterns

### Multi-Process System Modeling
```python
def build_integrated_system(query, processes):
    """Build integrated system from multiple processes"""
    
    models = {}
    
    # Create individual process models
    for process_name, process in processes.items():
        model = query.create_process_model(
            process,
            StageName=process_name
        )
        models[process_name] = model
    
    # Extend each model and link them together
    for name, model in models.items():
        query.extend_process(model)
        
        # Find linking opportunities
        for child in model.child_flows:
            if child.flow in [m.flow for m in models.values()]:
                # Link to internal process instead of background
                supplier = next(m for m in models.values() 
                              if m.flow == child.flow)
                child.terminate(supplier)
                print(f"Linked {child.flow} to internal model")
    
    return models
```

### Hierarchical Model Construction
```python
def create_hierarchical_model(query, main_process, detail_level=2):
    """Create hierarchical model with configurable detail"""
    
    # Create main model
    main_model = query.create_process_model(
        main_process,
        StageName='Main Process'
    )
    
    level = 1
    to_expand = [main_model]
    
    while to_expand and level <= detail_level:
        current_level = to_expand.copy()
        to_expand = []
        
        for model in current_level:
            # Expand this level
            query.extend_process(model, include_elementary=(level == detail_level))
            
            # Prepare next level expansions
            if level < detail_level:
                for child in model.child_flows:
                    if child.termination and hasattr(child.termination, 'inventory'):
                        # Create sub-model for significant processes
                        submodel = query.create_process_model(
                            child.termination,
                            StageName=f'{model["StageName"]} - {child.flow.name}'
                        )
                        child.terminate(submodel)
                        to_expand.append(submodel)
        
        level += 1
    
    return main_model
```

### Process Model Customization
```python
def customize_process_model(query, process, customizations):
    """Create process model with custom modifications"""
    
    # Create base model
    model = query.create_process_model(process)
    query.extend_process(model)
    
    # Apply customizations
    for custom in customizations:
        if custom['type'] == 'substitute_input':
            # Replace an input with alternative
            target_flow = custom['target_flow']
            replacement = custom['replacement']
            
            child = next(c for c in model.child_flows 
                        if c.flow == target_flow and c.direction == 'Input')
            child.terminate(replacement)
            print(f"Substituted {target_flow} with {replacement}")
            
        elif custom['type'] == 'adjust_efficiency':
            # Modify exchange values
            target_flow = custom['target_flow']
            factor = custom['factor']
            
            child = next(c for c in model.child_flows 
                        if c.flow == target_flow)
            old_value = child.exchange_value()
            query.observe(child, exchange_value=old_value * factor)
            print(f"Adjusted {target_flow} by factor {factor}")
            
        elif custom['type'] == 'add_input':
            # Add new input not in original process
            new_input = query.new_fragment(
                custom['flow'],
                'Input',
                parent=model,
                value=custom['value']
            )
            if 'termination' in custom:
                new_input.terminate(custom['termination'])
            print(f"Added new input: {custom['flow']}")
    
    return model
```

## Exchange Handling and Termination

### Automatic Termination Logic
```python
def analyze_termination_strategy(query, exchanges):
    """Understand how exchanges get terminated"""
    
    termination_types = {
        'context': 0,      # Elementary flows to environmental contexts
        'cutoff': 0,       # Unlinked intermediate flows  
        'process': 0,      # Linked to background processes
        'fragment': 0      # Linked to foreground fragments
    }
    
    for exchange in exchanges:
        if exchange.type == 'context':
            termination_types['context'] += 1
        elif exchange.type == 'cutoff':
            # Check if auto_anchor finds a fragment
            matching_frags = list(query.fragments_with_flow(
                exchange.flow, 
                exchange.direction
            ))
            if matching_frags:
                termination_types['fragment'] += 1
            else:
                termination_types['cutoff'] += 1
        else:  # process termination
            termination_types['process'] += 1
    
    print("Termination strategy analysis:")
    for term_type, count in termination_types.items():
        print(f"  {term_type}: {count} exchanges")
    
    return termination_types
```

### Custom Termination Control
```python
def create_model_with_termination_control(query, process, term_dict=None):
    """Create model with explicit termination control"""
    
    if term_dict is None:
        term_dict = {}
    
    # Get process inventory
    inventory = process.inventory()
    
    # Build termination dictionary for specific flows
    for exchange in inventory:
        flow_id = exchange.flow.external_ref
        
        if flow_id in term_dict:
            continue  # Already specified
            
        # Custom termination logic
        if 'electricity' in exchange.flow.name.lower():
            # Link electricity to specific source
            electricity_model = query.frag('renewable_electricity')
            term_dict[flow_id] = electricity_model
            
        elif 'transport' in exchange.flow.name.lower():
            # Link transport to specific service
            transport_model = query.frag('electric_transport')
            term_dict[flow_id] = transport_model
    
    # Create model with custom terminations
    model = query.fragment_from_exchanges(
        inventory,
        term_dict=term_dict,
        ref=f'{process.name}_custom_model',
        auto_anchor=False  # Rely on term_dict only
    )
    
    return model
```

## Process Model Validation

### Model Integrity Checks
```python
def validate_process_model(query, model):
    """Validate process model integrity"""
    
    issues = []
    
    # Check for unobserved fragments
    tree = query.tree(model)
    for node in tree:
        if not hasattr(node.fragment, 'exchange_value') or node.fragment.exchange_value() is None:
            issues.append(f"Unobserved fragment: {node.fragment.external_ref}")
    
    # Check for cutoff flows
    cutoffs = []
    for node in tree:
        if hasattr(node.fragment, 'termination'):
            term = node.fragment.termination()
            if term is None or term.is_null:
                cutoffs.append(node.fragment)
    
    if cutoffs:
        issues.append(f"Cutoff flows: {len(cutoffs)}")
    
    # Check balance
    try:
        traversal = query.traverse(model)
        total_flows = len(list(traversal))
        print(f"Model traverses successfully: {total_flows} flows")
    except Exception as e:
        issues.append(f"Traversal error: {e}")
    
    if issues:
        print("Model validation issues:")
        for issue in issues:
            print(f"  - {issue}")
    else:
        print("Model validation: PASSED")
    
    return len(issues) == 0
```

## Integration Best Practices

### Process Selection
- **Choose representative processes**: Select processes that best represent your specific technology
- **Consider system boundaries**: Ensure process scope matches your modeling needs
- **Verify data quality**: Check process documentation and data sources

### Model Structure
- **Use StageName consistently**: Organize models with clear stage naming conventions
- **Preserve process metadata**: Maintain links to original process documentation
- **Document customizations**: Record any modifications made to original process data

### Termination Strategy
- **Plan linking strategy**: Decide which flows to link internally vs. leave as cutoffs
- **Use auto_anchor judiciously**: Automatic linking can create unintended connections
- **Validate terminations**: Ensure terminations make sense from a system perspective

### Performance Considerations
- **Limit extension depth**: Deep process extensions can create very large models
- **Cache process models**: Reuse created models rather than recreating them
- **Use scenarios for alternatives**: Rather than creating multiple models, use scenarios

## Next Steps

- Master [Advanced Operations](../advanced-operations/) for model manipulation
- Learn [Scenarios](../scenarios/) for multi-variant modeling
- Explore [Balance Flows](../balance-flows/) for conservation constraints
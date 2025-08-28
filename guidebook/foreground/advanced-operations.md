---
layout: page
title: Advanced Fragment Operations
permalink: /guidebook/foreground/advanced-operations/
---

# Advanced Fragment Operations

The advanced foreground implementation provides sophisticated tools for manipulating and restructuring fragment models. These operations enable complex modeling workflows including model composition, decomposition, and optimization.

## Fragment Lifecycle Operations

### Fragment Cloning

#### `clone_fragment(fragment, tag=None, **kwargs)`
Create independent copies of fragment models.
```python
# Clone a complete model
base_steel_model = query.frag('steel_production_base')
electric_steel_model = query.clone_fragment(
    base_steel_model,
    tag='electric_arc'
)

# Clone creates independent copy with new UUIDs
print(f"Original: {base_steel_model.uuid}")
print(f"Clone: {electric_steel_model.uuid}")

# Tag modifies external_ref of named fragments
print(f"Original children: {[c.external_ref for c in base_steel_model.child_flows]}")
print(f"Clone children: {[c.external_ref for c in electric_steel_model.child_flows]}")
```

**Cloning behavior:**
- Creates completely independent fragment tree
- Generates new UUIDs for all fragments
- Preserves all observations and terminations
- Appends tag to named external_refs
- Maintains parent-child relationships

### Fragment Deletion

#### `delete_fragment(fragment, **kwargs)`
Remove fragments and their subtrees from the model.
```python
# Delete specific fragment and all children
obsolete_model = query.frag('old_process_model')
success = query.delete_fragment(obsolete_model)

if success:
    print("Fragment tree deleted successfully")
else:
    print("Fragment not found")

# Deletion is recursive - removes entire subtree
# Use with caution as there's no undo!
```

**Deletion behavior:**
- Removes fragment from archive
- Recursively deletes all child fragments
- Does NOT remove entities from memory (for safety)
- No safety checking - use with caution
- Returns `True` if successful, `False` if fragment not found

## Fragment Restructuring Operations

### Fragment Splitting

#### `split_subfragment(fragment, replacement=None, descend=False, **kwargs)`
Convert child fragments into independent reference fragments.
```python
# Example: Split packaging component into reusable module
widget_model = query.frag('widget_production')
packaging_child = next(c for c in widget_model.child_flows 
                      if 'packaging' in c.external_ref)

# Split without replacement - creates new reference fragment
packaging_module = query.split_subfragment(packaging_child)

print(f"New reference fragment: {packaging_module.external_ref}")
print(f"Widget model now links to: {packaging_child.termination}")

# Split with replacement - link to different fragment
alternative_packaging = query.frag('eco_packaging_module')
packaging_module = query.split_subfragment(
    packaging_child,
    replacement=alternative_packaging
)
```

**Split operation effects:**
```
Before: ...parent --> child_fragment
After:  ...parent --> surrogate # new_reference_fragment
                      (new_reference_fragment)

With replacement:
Before: ...parent --> child_fragment; (replacement_fragment)  
After:  ...parent --> surrogate # replacement_fragment
                      (child_fragment); (replacement_fragment)
```

### Fragment Interposition

#### `interpose(fragment, balance=True)`
Insert intermediate fragment for complex modeling.
```python
# Insert intermediate processing step
steel_input = query.get('steel-input-fragment')
processing_fragment = query.interpose(steel_input, balance=True)

print(f"Original path: parent -> steel_input")
print(f"New path: parent -> processing_fragment -> steel_input")

# Often used with balance flows for mass/energy accounting
if balance:
    steel_input.set_balance_flow()  # Automatically computed
```

**Interposition creates:**
- New intermediate fragment between parent and child
- Optional balance flow relationship
- Opportunity to model intermediate processing steps

## Entity Management Operations

### Smart Entity Creation

#### `add_or_retrieve(external_ref, reference, name, group=None, strict=False, **kwargs)`
Create entities only if they don't already exist.
```python
# Create flow if it doesn't exist
custom_steel = query.add_or_retrieve(
    external_ref='custom_steel_alloy',
    reference='kg',  # or quantity entity
    name='Custom Steel Alloy',
    group='Materials'
)

# Strict mode validates existing entities
try:
    verified_steel = query.add_or_retrieve(
        external_ref='custom_steel_alloy',
        reference='kg',
        name='Custom Steel Alloy',
        strict=True  # Will raise error if name/reference differs
    )
except (TypeError, ValueError) as e:
    print(f"Entity validation failed: {e}")

# Non-strict mode updates existing entities
flexible_steel = query.add_or_retrieve(
    external_ref='custom_steel_alloy',
    reference='kg',
    name='Updated Steel Alloy Name',
    strict=False  # Updates name if different
)
```

### Bulk Entity Import

#### `post_entity_refs(entity_refs, **kwargs)`
Import multiple entity references at once.
```python
# Import entities from external source
external_entities = [
    {'external_ref': 'material_1', 'name': 'Material 1', 'entity_type': 'flow'},
    {'external_ref': 'material_2', 'name': 'Material 2', 'entity_type': 'flow'},
    {'external_ref': 'process_1', 'name': 'Process 1', 'entity_type': 'process'}
]

# Create catalog refs and import
entity_refs = []
for entity_data in external_entities:
    ref = catalog.catalog_ref(
        'external_source', 
        entity_data['external_ref'],
        entity_type=entity_data['entity_type']
    )
    ref['Name'] = entity_data['name']
    entity_refs.append(ref)

query.post_entity_refs(entity_refs)
```

## Advanced Fragment Manipulation

### Complex Model Composition
```python
def compose_complex_model(query, base_models, integration_rules):
    """Compose complex model from multiple base models"""
    
    # Clone base models to avoid modifying originals
    composed_models = {}
    for name, model in base_models.items():
        composed_models[name] = query.clone_fragment(model, tag=f'composed_{name}')
    
    # Apply integration rules
    for rule in integration_rules:
        if rule['type'] == 'link_models':
            # Link output of one model to input of another
            source_model = composed_models[rule['source']]
            target_model = composed_models[rule['target']]
            connection_flow = rule['flow']
            
            # Find connection points
            output = next(c for c in source_model.child_flows 
                         if c.flow == connection_flow and c.direction == 'Output')
            input_frag = next(c for c in target_model.child_flows 
                             if c.flow == connection_flow and c.direction == 'Input')
            
            # Create connection
            input_frag.terminate(output)
            print(f"Linked {rule['source']} to {rule['target']} via {connection_flow}")
            
        elif rule['type'] == 'merge_stages':
            # Merge similar processing stages
            models_to_merge = [composed_models[m] for m in rule['models']]
            merged = merge_processing_stages(query, models_to_merge, rule['stage_name'])
            composed_models[rule['result_name']] = merged
    
    return composed_models
```

### Model Modularization
```python
def modularize_model(query, complex_model, module_specs):
    """Break complex model into reusable modules"""
    
    modules = {}
    
    for spec in module_specs:
        module_name = spec['name']
        target_flows = spec['flows']  # Flows that define the module boundary
        
        # Find fragment subtree for this module
        module_fragments = []
        for node in query.tree(complex_model):
            if node.fragment.flow in target_flows:
                module_fragments.append(node.fragment)
                # Include all descendants
                for child in query.tree(node.fragment):
                    module_fragments.append(child.fragment)
        
        # Create module by cloning relevant fragments
        if module_fragments:
            # Find top-most fragment in the module
            module_root = min(module_fragments, 
                            key=lambda f: len(list(query.tree(complex_model, root=f))))
            
            # Split out as independent module
            module = query.split_subfragment(module_root)
            modules[module_name] = module
            
            # Name the module appropriately
            query.observe(module, name=module_name)
            
            print(f"Created module: {module_name}")
    
    return modules
```

### Model Optimization Operations
```python
def optimize_model_structure(query, model):
    """Optimize model structure for performance and clarity"""
    
    optimizations = []
    
    # Find redundant branches
    tree = query.tree(model)
    flow_usage = {}
    
    for node in tree:
        flow = node.fragment.flow
        if flow not in flow_usage:
            flow_usage[flow] = []
        flow_usage[flow].append(node.fragment)
    
    # Identify flows used multiple times
    redundant_flows = {f: frags for f, frags in flow_usage.items() 
                      if len(frags) > 1}
    
    for flow, fragments in redundant_flows.items():
        # Check if these can be consolidated
        if all(f.direction == fragments[0].direction for f in fragments):
            # Same direction - potential for consolidation
            total_value = sum(f.exchange_value() or 0 for f in fragments)
            
            # Keep first fragment, update its value
            primary = fragments[0]
            query.observe(primary, exchange_value=total_value)
            
            # Delete redundant fragments
            for frag in fragments[1:]:
                query.delete_fragment(frag)
            
            optimizations.append(f"Consolidated {len(fragments)} uses of {flow}")
    
    # Find and eliminate zero-value branches
    zero_branches = []
    for node in query.tree(model):
        if hasattr(node.fragment, 'exchange_value'):
            if node.fragment.exchange_value() == 0:
                zero_branches.append(node.fragment)
    
    for branch in zero_branches:
        query.delete_fragment(branch)
        optimizations.append(f"Removed zero-value branch: {branch.external_ref}")
    
    return optimizations
```

## Model Validation and Quality Assurance

### Structural Validation
```python
def validate_model_structure(query, model):
    """Validate model structural integrity"""
    
    issues = []
    
    # Check for orphaned fragments
    tree = query.tree(model)
    for node in tree:
        if node.fragment.parent and node.fragment.parent not in [n.fragment for n in tree]:
            issues.append(f"Orphaned fragment: {node.fragment.external_ref}")
    
    # Check for circular references
    visited = set()
    def check_circular(fragment, path):
        if fragment in path:
            issues.append(f"Circular reference detected: {[f.external_ref for f in path]}")
            return
        if fragment in visited:
            return
        
        visited.add(fragment)
        path.append(fragment)
        
        for child in fragment.child_flows:
            check_circular(child, path.copy())
    
    check_circular(model, [])
    
    # Check for inconsistent directions
    for node in tree:
        if hasattr(node.fragment, 'parent') and node.fragment.parent:
            # Child inputs should generally correspond to parent consumption
            # This is domain-specific validation
            pass
    
    return issues
```

### Performance Analysis
```python
def analyze_model_performance(query, model):
    """Analyze model performance characteristics"""
    
    stats = {
        'total_fragments': 0,
        'depth': 0,
        'breadth': 0,
        'termination_types': {},
        'scenarios': set()
    }
    
    tree = list(query.tree(model))
    stats['total_fragments'] = len(tree)
    stats['depth'] = max(node.depth for node in tree) if tree else 0
    
    # Analyze breadth at each level
    levels = {}
    for node in tree:
        if node.depth not in levels:
            levels[node.depth] = 0
        levels[node.depth] += 1
    
    stats['breadth'] = max(levels.values()) if levels else 0
    
    # Analyze terminations
    for node in tree:
        if hasattr(node.fragment, 'termination'):
            term = node.fragment.termination()
            if term:
                term_type = term.__class__.__name__
                stats['termination_types'][term_type] = stats['termination_types'].get(term_type, 0) + 1
    
    # Collect scenarios
    for scenario in query.scenarios(model, recurse=True):
        stats['scenarios'].add(scenario)
    
    return stats
```

## Best Practices

### Fragment Operations
- **Test before production**: Always test complex operations on cloned models first
- **Document changes**: Keep records of model modifications for reproducibility
- **Validate after operations**: Run validation checks after structural changes

### Model Composition
- **Plan integration carefully**: Design integration rules before composing models
- **Preserve provenance**: Maintain links to original models and data sources
- **Test integrated models**: Validate that composed models behave correctly

### Performance Optimization
- **Monitor model size**: Large models can impact performance significantly
- **Use lazy loading**: Don't traverse entire models unless necessary
- **Cache results**: Store computed results to avoid repeated calculations

## Next Steps

- Explore [Scenarios](../scenarios/) for multi-variant modeling
- Learn [Balance Flows](../balance-flows/) for conservation constraints
- Master [Fragment Discovery](../discovery/) for efficient model navigation
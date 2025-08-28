---
layout: page
title: Balance Flows and Conservation
permalink: /guidebook/foreground/balance-flows/
---

# Balance Flows and Conservation

Balance flows are a sophisticated feature of the advanced foreground implementation that enforce mass and energy conservation constraints in fragment models. They enable automatic calculation of flows that maintain physical consistency in complex product systems.

## Core Balance Flow Concepts

### What Are Balance Flows?
Balance flows are **automatically computed exchanges** that ensure conservation of mass, energy, or other quantities within fragment models. Instead of being directly observed, balance flows are calculated to maintain the equality:

**Inputs + Outputs = 0** (for each conserved quantity)

### Balance Flow Characteristics
- **Auto-calculated**: Values computed automatically during model traversal
- **Conservation-driven**: Maintain material/energy balances  
- **Scenario-aware**: Recalculated for each scenario
- **Reference quantity specific**: Balance different quantities independently

### Setting Balance Flows

#### `set_balance_flow(fragment, **kwargs)`
Designate a fragment as a balance flow.
```python
# Create widget production model
widget_model = query.new_fragment(widget_flow, 'Output', value=1.0)

# Add material inputs
steel_input = query.new_fragment(
    steel_flow, 'Input', 
    parent=widget_model, value=2.5
)
plastic_input = query.new_fragment(
    plastic_flow, 'Input',
    parent=widget_model, value=0.8
)

# Create balance flow for waste output
waste_output = query.new_fragment(
    waste_flow, 'Output',
    parent=widget_model
)

# Set as balance flow - will be calculated automatically
query.set_balance_flow(waste_output)
print(f"Waste output is balance flow: {waste_output.is_balance}")
```

#### `unset_balance_flow(fragment, **kwargs)`
Remove balance flow designation.
```python
# Remove balance status - fragment retains last calculated value
query.unset_balance_flow(waste_output)
print(f"Waste output value: {waste_output.exchange_value()}")
```

## Balance Flow Patterns

### Mass Balance
```python
def create_mass_balanced_process(query, process_name, inputs, outputs):
    """Create process with automatic mass balance"""
    
    # Main product flow (assume mass quantity)
    main_product = outputs[0]  # First output is main product
    process_model = query.new_fragment(
        main_product['flow'], 'Output',
        value=main_product['value'],
        name=process_name
    )
    
    # Add specified inputs
    total_input_mass = 0
    for inp in inputs:
        input_frag = query.new_fragment(
            inp['flow'], 'Input',
            parent=process_model,
            value=inp['value']
        )
        # Calculate mass (assuming kg or compatible units)
        total_input_mass += inp['value']
    
    # Add specified outputs (except main product)
    total_output_mass = main_product['value']
    for out in outputs[1:]:
        output_frag = query.new_fragment(
            out['flow'], 'Output',
            parent=process_model,
            value=out['value']
        )
        total_output_mass += out['value']
    
    # Create balance flow if mass doesn't balance
    mass_difference = total_input_mass - total_output_mass
    if abs(mass_difference) > 0.001:  # Tolerance for rounding
        if mass_difference > 0:
            # Excess input mass -> waste output
            waste_flow = query.get_canonical_flow('process waste')
            balance_frag = query.new_fragment(
                waste_flow, 'Output',
                parent=process_model
            )
        else:
            # Deficit -> auxiliary input needed
            auxiliary_flow = query.get_canonical_flow('auxiliary input')
            balance_frag = query.new_fragment(
                auxiliary_flow, 'Input',
                parent=process_model
            )
        
        query.set_balance_flow(balance_frag)
        print(f"Created balance flow: {balance_frag.external_ref}")
    
    return process_model
```

### Energy Balance
```python
def create_energy_balanced_system(query, system_name, energy_inputs, energy_outputs, efficiency=0.9):
    """Create system with energy balance constraints"""
    
    # Main energy service output
    main_service = energy_outputs[0]
    system_model = query.new_fragment(
        main_service['flow'], 'Output',
        value=main_service['value'],
        name=system_name
    )
    
    # Energy inputs
    total_energy_input = 0
    for energy_in in energy_inputs:
        input_frag = query.new_fragment(
            energy_in['flow'], 'Input',
            parent=system_model,
            value=energy_in['value']
        )
        # Convert to common energy units (MJ)
        energy_content = energy_in['value'] * energy_in.get('energy_factor', 1.0)
        total_energy_input += energy_content
    
    # Specified energy outputs
    total_energy_output = main_service['value'] * main_service.get('energy_factor', 1.0)
    for energy_out in energy_outputs[1:]:
        output_frag = query.new_fragment(
            energy_out['flow'], 'Output',
            parent=system_model,
            value=energy_out['value']
        )
        energy_content = energy_out['value'] * energy_out.get('energy_factor', 1.0)
        total_energy_output += energy_content
    
    # Calculate energy balance with efficiency
    useful_energy = total_energy_input * efficiency
    energy_loss = useful_energy - total_energy_output
    
    if energy_loss > 0:
        # Create waste heat balance flow
        waste_heat = query.get_canonical_flow('waste heat')
        heat_balance = query.new_fragment(
            waste_heat, 'Output',
            parent=system_model
        )
        query.set_balance_flow(heat_balance)
    
    return system_model
```

## Advanced Balance Flow Applications

### Multi-Component Balance
```python
def create_multi_component_balance(query, reactor_model, components):
    """Create balance flows for multiple chemical components"""
    
    balance_flows = {}
    
    for component in components:
        component_name = component['name']
        conservation_rule = component.get('conservation', 'balance')
        
        if conservation_rule == 'balance':
            # Standard mass balance for this component
            balance_flow = query.new_fragment(
                component['balance_flow'], 'Output',
                parent=reactor_model
            )
            query.set_balance_flow(balance_flow)
            balance_flows[component_name] = balance_flow
            
        elif conservation_rule == 'consumed':
            # Component is completely consumed (no balance flow needed)
            pass
            
        elif conservation_rule == 'catalyst':
            # Catalytic component - input equals output
            catalyst_input = next(
                c for c in reactor_model.child_flows 
                if c.flow == component['flow'] and c.direction == 'Input'
            )
            catalyst_output = query.new_fragment(
                component['flow'], 'Output',
                parent=reactor_model,
                value=catalyst_input.exchange_value()
            )
            # Not a balance flow - fixed relationship
    
    return balance_flows
```

### Process Intensification Modeling
```python
def model_process_intensification(query, base_process, intensification_factor):
    """Model process intensification with balance flow adjustments"""
    
    # Clone base process for intensification modeling
    intensified_process = query.clone_fragment(
        base_process, 
        tag='intensified'
    )
    
    # Identify balance flows in the model
    balance_flows = []
    for node in query.tree(intensified_process):
        if hasattr(node.fragment, 'is_balance') and node.fragment.is_balance:
            balance_flows.append(node.fragment)
    
    # Apply intensification to non-balance flows
    for node in query.tree(intensified_process):
        if node.fragment not in balance_flows and hasattr(node.fragment, 'exchange_value'):
            current_value = node.fragment.exchange_value()
            if current_value:
                # Apply intensification factor
                new_value = current_value * intensification_factor
                query.observe(node.fragment, exchange_value=new_value)
    
    # Balance flows will automatically adjust during traversal
    print(f"Applied intensification factor {intensification_factor}")
    print(f"Balance flows will auto-adjust: {len(balance_flows)}")
    
    return intensified_process
```

## Balance Flow Validation and Analysis

### Conservation Checking
```python
def validate_conservation(query, model, quantities_to_check=None):
    """Validate conservation laws in fragment model"""
    
    if quantities_to_check is None:
        quantities_to_check = ['mass', 'energy']
    
    conservation_results = {}
    
    # Traverse model to get all flows
    traversal = query.traverse(model)
    
    for quantity_name in quantities_to_check:
        quantity = query.get_canonical(quantity_name)
        
        total_input = 0
        total_output = 0
        
        for flow in traversal:
            # Convert flow to check quantity units
            try:
                cf = query.cf(flow.fragment.flow, quantity)
                quantity_magnitude = flow.magnitude * cf
                
                if flow.fragment.direction == 'Input':
                    total_input += quantity_magnitude
                else:  # Output
                    total_output += quantity_magnitude
                    
            except Exception:
                # Skip flows that can't be converted to this quantity
                continue
        
        # Check balance
        imbalance = total_input - total_output
        relative_imbalance = abs(imbalance) / max(total_input, total_output) if max(total_input, total_output) > 0 else 0
        
        conservation_results[quantity_name] = {
            'total_input': total_input,
            'total_output': total_output,
            'imbalance': imbalance,
            'relative_imbalance': relative_imbalance,
            'balanced': relative_imbalance < 0.01  # 1% tolerance
        }
        
        print(f"{quantity_name.title()} Balance:")
        print(f"  Input: {total_input:.3f}")
        print(f"  Output: {total_output:.3f}")  
        print(f"  Imbalance: {imbalance:.3f} ({relative_imbalance:.1%})")
        print(f"  Balanced: {conservation_results[quantity_name]['balanced']}")
    
    return conservation_results
```

### Balance Flow Sensitivity
```python
def analyze_balance_sensitivity(query, model, parameter_changes):
    """Analyze how balance flows respond to parameter changes"""
    
    # Identify balance flows
    balance_flows = []
    for node in query.tree(model):
        if hasattr(node.fragment, 'is_balance') and node.fragment.is_balance:
            balance_flows.append(node.fragment)
    
    if not balance_flows:
        print("No balance flows found in model")
        return {}
    
    # Get baseline balance flow values
    baseline_traversal = query.traverse(model)
    baseline_balance_values = {}
    
    for flow in baseline_traversal:
        if flow.fragment in balance_flows:
            baseline_balance_values[flow.fragment.external_ref] = flow.magnitude
    
    sensitivity_results = {}
    
    # Test each parameter change
    for param_name, change_factor in parameter_changes.items():
        scenario_name = f'sensitivity_{param_name}_{change_factor}'
        
        # Find and modify parameter
        param = query.frag(param_name)  # Assuming named parameters
        base_value = param.exchange_value()
        query.observe(
            param,
            exchange_value=base_value * change_factor,
            scenario=scenario_name
        )
        
        # Traverse with modified parameter
        modified_traversal = query.traverse(model, scenario=scenario_name)
        
        # Compare balance flow changes
        param_results = {}
        for flow in modified_traversal:
            if flow.fragment in balance_flows:
                flow_name = flow.fragment.external_ref
                baseline_value = baseline_balance_values[flow_name]
                new_value = flow.magnitude
                
                change = new_value - baseline_value
                relative_change = change / baseline_value if baseline_value != 0 else float('inf')
                
                param_results[flow_name] = {
                    'baseline': baseline_value,
                    'new_value': new_value,
                    'absolute_change': change,
                    'relative_change': relative_change
                }
        
        sensitivity_results[param_name] = param_results
    
    return sensitivity_results
```

## Balance Flow Best Practices

### Design Principles
- **Physical realism**: Ensure balance flows represent physically meaningful quantities
- **Conservation scope**: Clearly define what quantity is being conserved (mass, energy, moles, etc.)
- **Boundary clarity**: Establish clear system boundaries for conservation calculations

### Implementation Guidelines
- **Single balance per system**: Generally use one balance flow per conservation law per system boundary
- **Meaningful flows**: Balance flows should represent real physical flows (waste, heat loss, etc.)
- **Validation**: Always validate that balance flows produce physically reasonable results

### Common Applications

#### Combustion Processes
```python
# Combustion with air balance
fuel_input = query.new_fragment(natural_gas, 'Input', parent=combustor, value=1.0)
air_input = query.new_fragment(air_flow, 'Input', parent=combustor)
query.set_balance_flow(air_input)  # Stoichiometric air requirement

co2_output = query.new_fragment(co2_flow, 'Output', parent=combustor, value=2.75)  # Stoichiometric CO2
h2o_output = query.new_fragment(water_flow, 'Output', parent=combustor, value=2.25)  # Stoichiometric H2O
```

#### Separation Processes
```python
# Distillation with material balance
feed_input = query.new_fragment(crude_feed, 'Input', parent=distillation, value=100.0)
overhead_output = query.new_fragment(light_product, 'Output', parent=distillation, value=30.0)
bottoms_balance = query.new_fragment(heavy_product, 'Output', parent=distillation)
query.set_balance_flow(bottoms_balance)  # Automatically calculated as 70.0
```

#### Chemical Reactions
```python
# Reaction with yield-based balance
reactant_a = query.new_fragment(chemical_a, 'Input', parent=reactor, value=50.0)
reactant_b = query.new_fragment(chemical_b, 'Input', parent=reactor, value=30.0)
main_product = query.new_fragment(product_c, 'Output', parent=reactor, value=60.0)  # 75% yield
byproduct_balance = query.new_fragment(byproduct_d, 'Output', parent=reactor)
query.set_balance_flow(byproduct_balance)  # Unreacted material + side products
```

## Troubleshooting Balance Flows

### Common Issues
1. **Negative balance flows**: May indicate modeling errors or unrealistic parameters
2. **Large balance flows**: Could suggest missing major flows or incorrect stoichiometry  
3. **Unstable balances**: May result from circular dependencies or conflicting constraints

### Debugging Strategies
```python
def debug_balance_flows(query, model):
    """Debug balance flow issues"""
    
    print("Balance Flow Debug Analysis")
    print("=" * 40)
    
    # Identify all balance flows
    balance_flows = []
    for node in query.tree(model):
        if hasattr(node.fragment, 'is_balance') and node.fragment.is_balance:
            balance_flows.append(node.fragment)
    
    print(f"Found {len(balance_flows)} balance flows:")
    for bf in balance_flows:
        print(f"  - {bf.external_ref}")
    
    # Check for common issues
    traversal = query.traverse(model)
    
    for flow in traversal:
        if flow.fragment in balance_flows:
            if flow.magnitude < 0:
                print(f"WARNING: Negative balance flow: {flow.fragment.external_ref} = {flow.magnitude}")
            
            if abs(flow.magnitude) > 1000:  # Threshold for "large" flows
                print(f"WARNING: Large balance flow: {flow.fragment.external_ref} = {flow.magnitude}")
    
    # Suggest conservation validation
    print("\nRecommendation: Run validate_conservation() to check mass/energy balance")
    
    return balance_flows
```

## Next Steps

- Review [Scenarios](../scenarios/) for balance flow behavior in different conditions
- Master [Advanced Operations](../advanced-operations/) for complex balance flow modeling  
- Explore [Process Models](../process-models/) for integrating balance flows with background data
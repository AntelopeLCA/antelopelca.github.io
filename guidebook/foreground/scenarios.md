---
layout: page
title: Scenario Management
permalink: /guidebook/foreground/scenarios/
---

# Scenario Management

The advanced foreground implementation provides sophisticated scenario management capabilities, enabling users to model alternative configurations, conduct sensitivity analyses, and compare different technology options within the same fragment model.

## Core Scenario Concepts

### What Are Scenarios?
Scenarios in Antelope foreground modeling represent **alternative model configurations** that can differ in:
- **Exchange values** (quantities of material/energy flows)
- **Terminations** (supply sources or technology choices)  
- **Process parameters** (efficiency factors, yield rates)

Each fragment can have multiple scenario-specific observations, allowing one model to represent multiple "what-if" cases.

### Scenario-Aware Observation

#### Enhanced `observe()` Method
The observe method supports scenario-specific observations:
```python
# Base case observation
steel_input = query.get('steel-input-fragment')
query.observe(steel_input, exchange_value=100.0, name='Steel Input')

# Scenario-specific observations
query.observe(
    steel_input,
    exchange_value=80.0,    # 20% efficiency improvement
    scenario='efficient_production'
)

query.observe(
    steel_input,
    exchange_value=120.0,   # Higher quality steel
    scenario='premium_grade'
)

# Scenario-specific terminations
renewable_steel = query.frag('renewable_steel_production')
query.observe(
    steel_input,
    anchor=renewable_steel,
    scenario='sustainable_sourcing'
)
```

## Scenario Discovery and Management

### Finding Available Scenarios

#### `scenarios(fragment, recurse=True, **kwargs)`
Discover all scenarios in a model.
```python
# Find scenarios in specific fragment
widget_model = query.frag('widget_production')
local_scenarios = list(query.scenarios(widget_model, recurse=False))
print(f"Fragment scenarios: {local_scenarios}")

# Find all scenarios in model tree
all_scenarios = list(query.scenarios(widget_model, recurse=True))
print(f"Model scenarios: {set(all_scenarios)}")
```

### Parameter Discovery

#### `knobs(search=None, **kwargs)`
Find adjustable parameters for scenario analysis.
```python
# Find all model parameters
parameters = list(query.knobs())
for param in parameters:
    print(f"Parameter: {param.external_ref}")
    print(f"  Default value: {param.exchange_value()}")
    
    # Check for scenario variants
    scenarios = list(query.scenarios(param, recurse=False))
    for scenario in scenarios:
        value = param.exchange_value(scenario)
        print(f"  {scenario}: {value}")

# Search for specific parameters
energy_params = list(query.knobs(search='energy'))
material_params = list(query.knobs(search='material'))
```

### Node Analysis by Scenario

#### `nodes(origin=None, scenario=None, **kwargs)`
Analyze model terminations for specific scenarios.
```python
# Analyze base case terminations
base_nodes = list(query.nodes())
print(f"Base case nodes: {len(base_nodes)}")

# Analyze scenario-specific terminations
renewable_nodes = list(query.nodes(scenario='renewable_energy'))
print(f"Renewable scenario nodes: {len(renewable_nodes)}")

# Compare termination differences
base_processes = {n.termination for n in base_nodes}
renewable_processes = {n.termination for n in renewable_nodes}

substitutions = renewable_processes - base_processes
print(f"Process substitutions in renewable scenario: {len(substitutions)}")
```

## Scenario-Based Analysis

### Model Traversal by Scenario

#### `traverse(fragment, scenario=None, **kwargs)`
Compute model flows for specific scenarios.
```python
# Base case traversal
widget_model = query.frag('widget_production')
base_traversal = query.traverse(widget_model)

# Scenario traversals  
efficient_traversal = query.traverse(widget_model, scenario='efficient_production')
premium_traversal = query.traverse(widget_model, scenario='premium_grade')

# Compare flow magnitudes
print("Flow comparison:")
for base_flow in base_traversal:
    efficient_flow = next((f for f in efficient_traversal 
                          if f.fragment == base_flow.fragment), None)
    
    if efficient_flow:
        change = (efficient_flow.magnitude - base_flow.magnitude) / base_flow.magnitude * 100
        print(f"{base_flow.fragment.external_ref}: {change:+.1f}%")
```

### Impact Assessment by Scenario

#### `fragment_lcia(fragment, quantity_ref, scenario=None, **kwargs)`
Calculate environmental impacts for different scenarios.
```python
# Compare climate impacts across scenarios
gwp_quantity = query.get_canonical('climate change')
scenarios_to_analyze = ['base', 'efficient_production', 'renewable_energy', 'premium_grade']

results = {}
for scenario in scenarios_to_analyze:
    if scenario == 'base':
        result = query.fragment_lcia(widget_model, gwp_quantity)
    else:
        result = query.fragment_lcia(widget_model, gwp_quantity, scenario=scenario)
    
    results[scenario] = result.total()
    print(f"{scenario}: {result.total():.2f} kg CO2-eq")

# Calculate relative changes
base_impact = results['base']
for scenario, impact in results.items():
    if scenario != 'base':
        change = (impact - base_impact) / base_impact * 100
        print(f"{scenario} vs base: {change:+.1f}%")
```

## Advanced Scenario Modeling

### Complex Scenario Construction
```python
def build_technology_scenarios(query, base_model):
    """Build comprehensive technology scenarios"""
    
    scenarios = {
        'baseline': {},
        'efficiency_improvements': {},
        'renewable_energy': {},  
        'circular_economy': {},
        'best_case': {}
    }
    
    # Define scenario parameters
    efficiency_improvements = {
        'energy_input': 0.8,      # 20% efficiency improvement
        'material_input': 0.9,    # 10% material efficiency
    }
    
    renewable_substitutions = {
        'electricity_input': query.frag('renewable_electricity'),
        'heat_input': query.frag('renewable_heat')
    }
    
    circular_economy_changes = {
        'steel_input': {'value': 0.7, 'anchor': query.frag('recycled_steel')},
        'plastic_input': {'value': 0.6, 'anchor': query.frag('bio_plastic')}
    }
    
    # Apply efficiency improvements scenario
    for param_name, factor in efficiency_improvements.items():
        param = query.frag(param_name)  # Assuming named parameters
        base_value = param.exchange_value()
        query.observe(
            param,
            exchange_value=base_value * factor,
            scenario='efficiency_improvements'
        )
    
    # Apply renewable energy scenario
    for input_name, renewable_source in renewable_substitutions.items():
        input_frag = query.frag(input_name)
        query.observe(
            input_frag,
            anchor=renewable_source,
            scenario='renewable_energy'
        )
    
    # Apply circular economy scenario
    for input_name, changes in circular_economy_changes.items():
        input_frag = query.frag(input_name)
        
        if 'value' in changes:
            base_value = input_frag.exchange_value()
            query.observe(
                input_frag,
                exchange_value=base_value * changes['value'],
                scenario='circular_economy'
            )
        
        if 'anchor' in changes:
            query.observe(
                input_frag,
                anchor=changes['anchor'],
                scenario='circular_economy'
            )
    
    # Combine all improvements for best case
    for param_name, factor in efficiency_improvements.items():
        param = query.frag(param_name)
        base_value = param.exchange_value()
        query.observe(
            param,
            exchange_value=base_value * factor,
            scenario='best_case'
        )
    
    for input_name, renewable_source in renewable_substitutions.items():
        input_frag = query.frag(input_name)
        query.observe(
            input_frag,
            anchor=renewable_source,
            scenario='best_case'
        )
    
    return list(scenarios.keys())
```

### Sensitivity Analysis Framework
```python
def perform_sensitivity_analysis(query, model, parameters, quantity, ranges=None):
    """Perform systematic sensitivity analysis"""
    
    if ranges is None:
        ranges = {}
    
    base_result = query.fragment_lcia(model, quantity)
    base_impact = base_result.total()
    
    sensitivity_results = {}
    
    for param in parameters:
        param_name = param.external_ref
        base_value = param.exchange_value()
        
        # Define parameter range
        if param_name in ranges:
            test_range = ranges[param_name]
        else:
            # Default ±20% range
            test_range = [0.8, 0.9, 1.0, 1.1, 1.2]
        
        param_results = []
        
        for multiplier in test_range:
            scenario_name = f'sensitivity_{param_name}_{multiplier}'
            test_value = base_value * multiplier
            
            # Set parameter value for this test
            query.observe(
                param,
                exchange_value=test_value,
                scenario=scenario_name
            )
            
            # Calculate impact
            result = query.fragment_lcia(model, quantity, scenario=scenario_name)
            impact = result.total()
            
            # Calculate sensitivity
            param_change = (multiplier - 1.0) * 100  # Percentage change in parameter
            impact_change = (impact - base_impact) / base_impact * 100  # Percentage change in impact
            
            sensitivity = impact_change / param_change if param_change != 0 else 0
            
            param_results.append({
                'multiplier': multiplier,
                'parameter_value': test_value,
                'impact': impact,
                'parameter_change_pct': param_change,
                'impact_change_pct': impact_change,
                'sensitivity': sensitivity
            })
        
        sensitivity_results[param_name] = param_results
    
    return sensitivity_results
```

### Monte Carlo Scenario Generation
```python
import random

def generate_monte_carlo_scenarios(query, model, parameters, num_scenarios=100):
    """Generate Monte Carlo scenarios for uncertainty analysis"""
    
    # Define parameter uncertainty distributions
    param_distributions = {}
    for param in parameters:
        base_value = param.exchange_value()
        # Assume normal distribution with 10% coefficient of variation
        param_distributions[param.external_ref] = {
            'mean': base_value,
            'std': base_value * 0.1,
            'type': 'normal'
        }
    
    # Generate scenarios
    scenarios = []
    for i in range(num_scenarios):
        scenario_name = f'monte_carlo_{i:03d}'
        scenario_params = {}
        
        for param in parameters:
            param_name = param.external_ref
            distribution = param_distributions[param_name]
            
            if distribution['type'] == 'normal':
                # Sample from normal distribution
                value = random.normalvariate(distribution['mean'], distribution['std'])
                # Ensure positive values
                value = max(0.01 * distribution['mean'], value)
            
            scenario_params[param_name] = value
            
            # Apply to model
            query.observe(
                param,
                exchange_value=value,
                scenario=scenario_name
            )
        
        scenarios.append({
            'name': scenario_name,
            'parameters': scenario_params
        })
    
    return scenarios
```

## Scenario Analysis Workflows

### Comparative Assessment
```python
def compare_scenarios(query, model, scenarios, impact_methods):
    """Compare multiple scenarios across multiple impact categories"""
    
    results_matrix = {}
    
    # Initialize results structure
    for scenario in scenarios:
        results_matrix[scenario] = {}
    
    # Calculate impacts for each scenario and method
    for method_name, quantity in impact_methods.items():
        print(f"Calculating {method_name}...")
        
        for scenario in scenarios:
            if scenario == 'base':
                result = query.fragment_lcia(model, quantity)
            else:
                result = query.fragment_lcia(model, quantity, scenario=scenario)
            
            results_matrix[scenario][method_name] = result.total()
    
    # Generate comparison report
    print("\nScenario Comparison Results:")
    print("-" * 80)
    
    # Header
    header = f"{'Scenario':<20}"
    for method in impact_methods.keys():
        header += f"{method:<15}"
    print(header)
    
    # Results
    for scenario in scenarios:
        row = f"{scenario:<20}"
        for method_name in impact_methods.keys():
            impact = results_matrix[scenario][method_name]
            row += f"{impact:<15.2f}"
        print(row)
    
    # Relative changes vs base
    if 'base' in scenarios:
        print("\nRelative Changes vs Base (%):")
        print("-" * 80)
        
        header = f"{'Scenario':<20}"
        for method in impact_methods.keys():
            header += f"{method:<15}"
        print(header)
        
        base_results = results_matrix['base']
        for scenario in scenarios:
            if scenario == 'base':
                continue
                
            row = f"{scenario:<20}"
            for method_name in impact_methods.keys():
                base_impact = base_results[method_name]
                scenario_impact = results_matrix[scenario][method_name]
                change_pct = (scenario_impact - base_impact) / base_impact * 100
                row += f"{change_pct:<15.1f}"
            print(row)
    
    return results_matrix
```

### Scenario Optimization
```python
def optimize_scenarios(query, model, objective_quantity, constraints=None):
    """Find optimal scenario configurations"""
    
    if constraints is None:
        constraints = {}
    
    # Get all adjustable parameters
    parameters = list(query.knobs())
    
    # Define optimization space
    param_ranges = {}
    for param in parameters:
        base_value = param.exchange_value()
        param_ranges[param.external_ref] = {
            'min': base_value * 0.5,   # 50% reduction
            'max': base_value * 1.5,   # 50% increase  
            'base': base_value
        }
    
    # Simple grid search optimization (can be replaced with more sophisticated methods)
    best_scenario = None
    best_impact = float('inf')
    
    test_points = 5  # Points to test per parameter
    total_combinations = test_points ** len(parameters)
    
    print(f"Testing {total_combinations} scenario combinations...")
    
    combination_count = 0
    for param_values in generate_parameter_combinations(param_ranges, test_points):
        combination_count += 1
        scenario_name = f'optimization_{combination_count:04d}'
        
        # Apply parameter values
        for param in parameters:
            param_name = param.external_ref
            value = param_values[param_name]
            query.observe(param, exchange_value=value, scenario=scenario_name)
        
        # Check constraints
        if constraints:
            constraint_satisfied = True
            for constraint_quantity, max_impact in constraints.items():
                result = query.fragment_lcia(model, constraint_quantity, scenario=scenario_name)
                if result.total() > max_impact:
                    constraint_satisfied = False
                    break
            
            if not constraint_satisfied:
                continue
        
        # Calculate objective
        result = query.fragment_lcia(model, objective_quantity, scenario=scenario_name)
        impact = result.total()
        
        if impact < best_impact:
            best_impact = impact
            best_scenario = {
                'name': scenario_name,
                'parameters': param_values.copy(),
                'impact': impact
            }
        
        if combination_count % 100 == 0:
            print(f"Tested {combination_count}/{total_combinations} combinations...")
    
    return best_scenario

def generate_parameter_combinations(param_ranges, points_per_param):
    """Generate parameter value combinations for optimization"""
    import itertools
    
    param_names = list(param_ranges.keys())
    value_lists = []
    
    for param_name in param_names:
        range_info = param_ranges[param_name]
        values = []
        for i in range(points_per_param):
            # Linear interpolation between min and max
            fraction = i / (points_per_param - 1) if points_per_param > 1 else 0
            value = range_info['min'] + fraction * (range_info['max'] - range_info['min'])
            values.append(value)
        value_lists.append(values)
    
    # Generate all combinations
    for combination in itertools.product(*value_lists):
        yield dict(zip(param_names, combination))
```

## Best Practices

### Scenario Design
- **Use meaningful names**: Choose descriptive scenario names that indicate their purpose
- **Document assumptions**: Record the rationale behind scenario parameter choices
- **Validate scenarios**: Ensure scenario parameters represent realistic conditions

### Performance Considerations
- **Limit scenario proliferation**: Too many scenarios can impact model performance
- **Clean up test scenarios**: Remove temporary scenarios after analysis
- **Cache results**: Store scenario results to avoid repeated computation

### Analysis Quality
- **Use consistent baselines**: Always compare scenarios against the same baseline
- **Check parameter interactions**: Consider how different parameters might interact
- **Validate results**: Verify that scenario results make physical and logical sense

## Next Steps

- Learn [Balance Flows](../balance-flows/) for conservation constraints
- Master [Fragment Discovery](../discovery/) for efficient parameter identification
- Explore [Process Models](../process-models/) for scenario-aware background integration
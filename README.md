# Process Optimization: Heat Exchanger Network Lifecycle Costing

An automated MATLAB framework executing an Economic Pinch Analysis Loop. The engine evaluates 39 distinct process design configurations (Delta T_min from 2°C to 40°C) to dynamically minimize the Total Cost of Ownership across a 10-year factory lifecycle.

# Core Sizing & Financial Findings
* Optimal Temperature Approach (Delta T_min): 18 °C
* Minimum Steam Utility Target (Q_hot): 1,400.0 kW
* Minimum Water Utility Target (Q_cold): 0.0 kW (Achieves a physical Threshold Problem state)
* One-Time Equipment Capital Cost (CapEx): $483,877.53 USD
* Annual Plant Utility Expenditure (OpEx): $588,000.00 USD/year
* Optimized 10-Year Lifecycle Cost: $6,363,877.53 USD

# Framework Structure
The architecture wraps two core engineering methodologies into a single iterative loop:
1. Phase 2 (Thermodynamic Engine): Implements the industrial Problem Table Algorithm (heat cascade logic) to isolate distinct shifted temperature intervals and compute minimum hot/cold utility boundaries.
2. Phase 3 (Economic Evaluator): Applies a non-linear power-law costing formula (*Cost = C_{fixed} + C_{area} \cdot Area^{0.8}$) to reflect a realistic equipment economy of scale, automatically identifying the lowest lifecycle cost point.

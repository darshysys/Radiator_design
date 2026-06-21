# FSAE Electric Vehicle Radiator Design using Epsilon-NTU Thermal Model

A MATLAB Live Script tool for sizing a liquid-to-air radiator that cools the motor and motor controllers of a Formula Student electric vehicle, using the Epsilon-NTU heat exchanger method.

## Problem Statement

The radiator must reject **5.5 kW** of peak heat load from the drivetrain under the worst realistic thermal condition: maximum motor current draw during low-speed acceleration, when there is little to no ram air assisting the cooling fan. Coolant enters the radiator at a maximum of **60°C** and is pure water.

Rather than hand-calculating a single design point, this tool sweeps the key design variables - fin area density, radiator face area, and coolant flow rate - across realistic ranges and visualises the entire feasible design space, so the final geometry can be chosen based on real packaging and manufacturing constraints rather than a single guess.

## Design Methodology Summary

1. **Boundary conditions** are set from the worst-case operating point: fan-only airflow (no ram air), 25°C ambient air, 1.5 m/s face velocity (conservative low end of typical FSAE fan performance).
2. **Coolant outlet temperature** is derived from `Q = ṁCpΔT` rather than assumed, giving ≈53.4°C for a 200 g/s flow rate.
3. **C_min identification**: for any realistic FSAE radiator face area, the air-side heat capacity rate (C_air) is far smaller than the water-side (C_water), so air is C_min. This is verified analytically rather than assumed.
4. **Heat transfer coefficients** are calculated independently for each side, laminar flat-plate correlation for air, Dittus-Boelter for water in flat tubes combined into an overall U. A sensitivity check shows U is dominated almost entirely by the air side (h_air ≈ 26 W/m²K vs h_water ≈ 500–3000 W/m²K), so coolant-side flow details barely move U.
5. **Full crossflow ε-NTU equation** (not the simplified C*≈0 case) is used in the MATLAB model for accuracy.
6. **Parameter sweep** across fin area density (σ) and face area (A_face) produces a 2D map of predicted heat rejection, from which a feasibility boundary (Q = 5500 W) and a design-margin boundary (Q = 6050 W, S.F=1.1) are extracted.
7. **Sensitivity sweeps** on radiator height (H) and coolant flow rate (ṁ) isolate the effect of each, independent of the main geometry sweep.

## Key Design Decisions & Why They Were Made

**Core depth (`d`) and height (`H`) are fixed inputs, not solved variables.**
Unlike face area and fin density, depth and height are set as semi-fixed parameters (with discrete options/dropdowns in the Live Script) rather than continuously optimised. This reflects how these dimensions are actually constrained in practice, by standard manufactured core thicknesses and by chassis/packaging limits given by the rest of the team — rather than by the thermal physics itself. A sensitivity sweep on each confirms this choice is reasonable:
- Depth: h_air ∝ L^0.5, so even a ~28% change in depth only shifts h_air by ~13%.
- Height: varying H from 300–400mm changes predicted Q by less than 0.5%, since H only affects performance indirectly through the coolant-side heat transfer coefficient.

**Tube width equals core depth.**
A flat tube's width dimension runs in the air-flow (depth) direction of the core, so `b_tube = d` by construction.

**U is calculated once, outside the main sweep loop.**
Since h_air depends only on face velocity and depth (both fixed for a given run), and h_water barely influences U (for the first iteration coolant flow rate is fixed at 200g/s so h_water is also fixed), the overall heat transfer coefficient is constant across the entire (σ, A_face) sweep.

**Coolant flow rate is treated as a free/explorable parameter, not fixed.**
Pump selection is a real trade-off (weight, electrical load vs. cooling margin), so flow rate is swept independently rather than treated as a constant. The resulting curve reveals a capacity-rate crossover: below ~84 g/s, water is the bottleneck (C_min) and additional flow rate helps heat rejection a lot; above that point, air becomes the permanent bottleneck and further flow increases give strongly diminishing returns. This directly informs minimum viable pump sizing.

**Width is derived, not independently swept.**
Since `A_face = W × H` and H is fixed per run, width is fully determined once A_face is chosen, it is not a free third dimension. Plotting required width across the (σ, A_face) space (with the Q=5500W boundary overlaid) turns the thermal feasibility plot into a direct packaging decision tool: higher fin density buys a narrower radiator for the same thermal performance, at the cost of pressure drop, fan power, and fouling risk.

## Known Simplifications & Limitations

- Wall conduction resistance is neglected.
- Face velocity is treated as constant regardless of fin spacing; in reality tighter fin pitch locally accelerates air through the core (mass conservation), which this model does not capture.
- Fin area density (σ) is treated as an independent geometric parameter rather than derived from a specific manufactured fin pitch/louvre geometry.
- All correlations (flat-plate laminar, Dittus-Boelter) carry their own inherent uncertainty (~10-15%), which is part of the justification for the 10% design margin used.
- The model assumes steady-state operation at the full 5.5kW load; real acceleration events are shorter than the time needed to reach this equilibrium, meaning actual peak design requirements may be somewhat conservative. Also with wind blowing, the radiator performance is only boosted. 

## Repository Structure

```
radiator_design.mlx      — Main MATLAB Live Script (all calculations & plots)
README.md                — This file
```

## How to Use

1. Open `radiator_design.mlx` in MATLAB (R2020a or later recommended for Live Script features).
2. Run the script from the top (`Run`, not section-by-section) to avoid stale variable issues between sections.
3. Adjust the dropdown values for core depth (`d`) and height (`H`) as packaging constraints are confirmed.
4. Use the contour plots to read off a feasible (σ, A_face) design point, then check the corresponding required width plot to confirm it fits your packaging envelope. `sigma_query` can be adjusgted according to need and the corresponding `A_face` can be obtained. 
5. Use the coolant flow rate sensitivity plot to confirm your chosen pump provides adequate margin above the capacity-rate crossover point.

## Outputs

- Contour map of predicted heat rejection Q across fin area density and face area, with Q=5500W (minimum) and Q=6050W (+10% margin) boundaries highlighted.
- Contour map of required radiator width across the same design space, for direct packaging trade-off analysis.
- Sensitivity plot of Q vs. radiator height (300–400mm range).
- Sensitivity plot of Q vs. coolant mass flow rate (50–400 g/s range), showing the capacity-rate crossover behaviour.

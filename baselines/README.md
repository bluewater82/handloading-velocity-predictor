# Baseline Models & Diagnostic Notebooks

This directory contains the **early baseline and diagnostic models** developed during the initial phase of the handloading / muzzle-velocity prediction project.

These notebooks are **not intended to be high-accuracy predictive models**.  
They are deliberately simple, underfit, and physically interpretable. Their purpose is to:

- Validate data integrity and units
- Verify that basic physical relationships behave correctly
- Detect data leakage and confounding early
- Establish reference error floors before adding complexity

Once validated, these baselines are **frozen** and retained as part of the project’s development record.

---

## Baseline 00 — Null Model

**Notebook:** `baseline_00_null_model.ipynb`

### Purpose
Establish a **minimum performance floor** by predicting the same value for every observation.

### Model
- Prediction: mean muzzle velocity of the dataset
- No features used

### Why it matters
Any future model must outperform this baseline to demonstrate that it has learned *something* beyond ignorance.  
This notebook provides a stable reference for evaluating later improvements and for detecting suspiciously strong “shortcut” features.

---

## Baseline 01 — Physics Sanity Check

**Notebook:** `baseline_01_physics_sanity.ipynb`

### Purpose
Verify that the dataset obeys **basic internal ballistics physics** within a single cartridge.

### Scope
- Cartridge-specific (e.g. .300 Blackout)
- Minimal feature set

### Model2
velocity_fps ~ charge_weight + bullet_weight


### Expected behavior
- `charge_weight` → positive coefficient  
- `bullet_weight` → negative coefficient  

Only the **direction** of effects is evaluated at this stage, not magnitude or accuracy.

### Why it matters
This baseline confirms:
- Units are consistent
- Features are wired correctly
- No obvious leakage or data corruption exists
- The dataset is suitable for further modeling

---

## Baseline 02 — Geometry Probe

**Notebook:** `baseline_02_geometry_probe.ipynb`

### Purpose
Test whether **barrel length** behaves sensibly once basic mass and charge effects are accounted for.

### Scope
- Same cartridge as Baseline 01
- Subset of data where barrel length is known

### Model3
velocity_fps ~ charge_weight + bullet_weight + barrel_length


### Interpretation notes
This baseline is intentionally sensitive to confounding.  
If barrel length shows weak or incorrect directionality, it indicates that:
- the effect is not identifiable in the current dataset without additional controls, or
- other omitted variables (e.g. powder chemistry) dominate the signal.

Such results are **informative**, not failures, and guide the introduction of the next feature.

---

## Design Philosophy

These baselines prioritize:
- Interpretability over performance
- Causal reasoning over correlation
- Early detection of leakage and bias

More expressive models and richer feature sets are introduced **only after** these checks pass.

This approach mirrors standard practice in serious scientific and industrial modeling workflows and prevents misleading downstream results.

---

## Next Steps
Subsequent baselines introduce **controlled chemical context** (e.g. burn rate index) and, later, regime-aware and multi-cartridge modeling.

See the main project documentation for current modeling status.

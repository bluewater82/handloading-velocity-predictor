# Muzzle Velocity Prediction from Handloading Data  
*A Gradient-Boosted Tree Modeling Study*

## Overview

This project develops a machine-learning model to predict **rifle muzzle velocity (fps)** from publicly available handloading data.  
Rather than pursuing accuracy alone, the project emphasizes **model comparison, physical plausibility, interpretability, and honest error analysis** across multiple rifle cartridges.

The final Version 1.0 model uses **gradient-boosted decision trees**, selected after systematic evaluation of linear, regularized linear, and nonlinear approaches.

---

## Problem Statement

Given a specific handload configuration, estimate the expected muzzle velocity while:

- avoiding data leakage,
- respecting known ballistic behavior,
- handling cross-cartridge heterogeneity,
- and maintaining defensible performance metrics.

This is a challenging regression problem due to:
- mixed data sources and manufacturers,
- non-linear internal ballistics,
- cartridge-specific operating regimes,
- and unavoidable real-world noise.

---

## Dataset & Features

The dataset aggregates manufacturer-published load data across multiple rifle cartridges.

### Target
- `velocity_fps`

### Numerical Features
- `charge_weight`
- `bullet_weight`
- `burn_rate_index` (ordinal powder burn-rate ranking)
- `barrel_length`

### Categorical Features
- `cartridge` (one-hot encoded)

Cartridges with mixed operational regimes (notably **.300 Blackout**) are intentionally excluded from the global model and addressed separately.

---

## Modeling Approach

### Baseline Phase
Early baselines used:
- mean predictors,
- ordinary least squares regression,
- and ridge regression with hand-engineered interactions.

These models served to:
- validate data integrity,
- confirm physically sensible feature behavior,
- detect cross-cartridge residual structure,
- and establish reference error floors.

### Final Model — Gradient-Boosted Trees

The final model uses **Histogram-based Gradient Boosting Regression** (`HistGradientBoostingRegressor`) in a scikit-learn pipeline.

Key characteristics:
- no feature scaling (tree-based model),
- automatic learning of nonlinear interactions,
- cross-validated hyperparameter tuning,
- MAE-focused loss function.

Tree models were selected because they naturally handle:
- cartridge-specific regimes,
- threshold effects,
- and nonlinear feature interactions that linear models required manual intervention to approximate.

---

## Performance (Version 1.0 — Global Model)

Evaluated on a held-out test set:

| Metric | Value |
|------|------|
| **MAE** | **44.5 fps** |
| **RMSE** | **59.9 fps** |
| **R²** | **0.963** |

These values are close to the expected noise floor of mixed-source ballistic data and outperform all linear and regularized baselines.

---

## Cartridge-Wise Performance

Mean Absolute Error by cartridge (fps):

- .308 Win: ~34  
- .224 Valkyrie: ~36  
- 6.5 Creedmoor: ~39  
- .300 Win Mag: ~41  
- .223 Rem: ~48  
- 6mm ARC: ~49  
- .260 Rem: ~53  
- 7mm Rem Mag: ~59  
- .243 Win: ~72  
- .270 Weatherby Mag: ~79  
- .270 Win: ~84  

Error increases smoothly with data sparsity and ballistic extremity.  
No cartridge collapses or exhibits suspiciously low error, indicating good generalization rather than memorization.

---

## Handling .300 Blackout

.300 Blackout was **explicitly excluded** from the global model.

Reason:
- mixed subsonic/supersonic regimes,
- wide barrel-length variability,
- pressure-velocity behavior distinct from standard rifle cartridges.

Including it degraded global performance and obscured model interpretation.  
A **dedicated sub-model** is planned to address .300 Blackout as a regime-specific problem.

---

## Design Philosophy

This project prioritizes:

- correctness over convenience,
- interpretability over blind optimization,
- domain knowledge over metric chasing,
- and honest reporting of limitations.

Models are only made more expressive after simpler approaches are shown to be insufficient.

---

## Limitations & Future Work

- Develop a regime-aware sub-model for .300 Blackout
- Expand barrel-length coverage for under-represented cartridges
- Add uncertainty estimation / prediction intervals
- Investigate physics-informed hybrid features

---

## Status

**Version 1.0 complete.**  
The current model is frozen and serves as the reference implementation for future extensions.

---

## Repository Structure

- baselines
- data
- models
  - v1_global_tree
  - v1_300blk

---

## License & Data

This project uses publicly available manufacturer data.  
No proprietary load data is included.

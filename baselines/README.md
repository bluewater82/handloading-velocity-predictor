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

### Model0
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

### Model1
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

### Model2
velocity_fps ~ charge_weight + bullet_weight + barrel_length


### Interpretation notes
This baseline is intentionally sensitive to confounding.  
If barrel length shows weak or incorrect directionality, it indicates that:
- the effect is not identifiable in the current dataset without additional controls, or
- other omitted variables (e.g. powder chemistry) dominate the signal.

Such results are **informative**, not failures, and guide the introduction of the next feature. For now, barrel length will not be considered in the next few iterations until more substantial barrel length data is obtained.

---

## Baseline 03 - Testing burn rates

**Notebook:** `baseline_03_burn_validation.ipynb`

### Purpose
Test whether the burn rate index will further improve prediction accuracy of the model.

### Scope
- Same cartridge
- Abandoning the barrel length feature due to insufficient variability with the cartridge
- Including burn rate ranking scheme to set of core features. The values are positional rankings (ordinal) and this test is meant to see how the model will interpret the ordinal values as opposed to actual measured units such as the mass for the bullet_weight and charge_weight features.

### Model3
velocity_fps ~ charge_weight + bullet_weight + burn_rate_index

### Interpretation notes
Unlike Model3's results which were confounded by adding the limited barrel length, Model4 showed another significant improvement in lowering MAE.
- Model's MAE dropped to 51.51ft/s, only 27% of the floor MAE of 191.95ft/s, showing that comparing burn rates provides a strong signal that meaningfully affects the accuracy of the model.
- Residuals are still structureless and non-biased, providing comfort that the model is still behaving

---

## Baseline 04 - Including cartridge geometry as a feature

**Notebook:** `baseline_04_coal.ipynb`

### Purpose
This baseline will test to see if including the cartridge's overall length (COAL) in the training data will yield meaningful results.

### Scope
- Retaining the same features as Model4 from the previous baseline test
- Expanded feature vector to include COAL measurements

### Model4
velocity_fps ~ charge_weight + bullet_weight + burn_rate_index + coal

### Interpretation notes
This model yielded interesting results that I did not expect. The COAL feature appeared at face-value to have a meaningful weight but provided virtually zero influence on the model's predictions. I assume this is due to the variations in COAL values being of extremely small magnitude - on the order of hundredths/thousandths of an inch.

At this stage I feel that the simple linear regression models have reached their limitations. The next major step will be to move to using Ridge models, but before I do I will rerun all previous exploratory base models with train_test_split. Those should be nearly identical to these previous baselines so I will not be adding notes here in the README unless I see something meaningfully different. I will still upload all new train/test baselines alongside their corresponding notebooks, marked as "_v2" in their file names for easy identification.

---

## Baseline 05 - Introduction of Ridge regression model

**Notebook:** `baseline_05_ridge.ipynb`

### Purpose
This will be the first model to use Ridge regression as opposed to OLS regression. Previous baseline models handled the core feature sets well but predictive accuracy stalled once we tried stepping away from that core set. These next few models will explore how Ridge handles the predictions.

### Scope
- Retaining same feature set as previous baseline model (04) to compare mean absolute error margins and residuals between OLS and Ridge before moving on to a more expansive training of the Ridge models.
- Still limiting this baseline training data to just one cartridge (.308 Winchester).
- This baseline is mostly a sanity check to make sure Ridge is behaving and picking up exactly where the OLS regression models left off. Given that the training data is identical, results are expected to be virtually identical to baseline_04.

### Model5
velocity_fps ~ charge_weight + bullet_weight + burn_rate_index + coal

### Interpretation notes
As expected, the introduction of Ridge regression did not yield any meaningfully-different results. Floor MAE remaining identical (190.84ft/s) and the model's predictive MAE was virtually the same as Model4's (51.18ft/s vs 51.21ft/s, respectively).
With this sanity check passed, we will start widenening the scope of the training data for future Ridge models.

---

## Baseline 06 - Training the model with categorical features

**Notebook:** `baseline_06_cats.ipynb`

### Purpose
- This model will use OneHotEncoding on a single categorical feature (cartridge type) to assess how such a feature impacts prediction behavior. Up until now, all training features were numerical. This addition of categorical features, even if it is just a single feature for this baseline, marks a definitive evolutionary step in the system's development.

### Scope
- Retaining core features (same as those used in previous baseline) and adding catridge designation as our new categorical feature.
- Full existing dataset will be evaluated for the first time. Previous baselines were limited to a single cartridge (.308 Winchester) in order to preserve physical identifiability of feature effects and avoid confouding introduced by the geometric and behavioral differences betwee cartridge types. This model will be assessing all cartridges that are currently existing in the curated dataset (.223 Remington, .243 Winchester, .260 Remington, .300 Blackout, .308 Winchester, 6.5 Creedmoor, 7mm Remington Magnum, .270 Winchester, .270 Weatherby Magnum, and .260 Remington).

### Model6
velocity_fps ~ cartridge + charge_weight + bullet_weight + burn_rate_index + coal

### Interpretation notes
- Our floor mean absolute error increased from 190ft/s to 269ft/s. This change in floor mean was to be expected since we introduced a significantly larger and wider set of velocities that spanned both subsonic and supersonic values. Until the current dataset grows with new cartridge loads, this will now be our new standard floor MAE.
- The model's MAE increased for the first time in the training models (51ft/s to 86ft/s). At first I believed this to be due to a breaking of the model's ability to meaningfully interpret the data and that the addition of our first categorical feature made the predictions worse instead of better.
- Secondary review of the results led to the realization that:
  - Our previous floor-to-model MAE reduction was 190ft/s -> 51ft/s, an improvement of roughly 139ft/s from ignorance
  - Our new floor-to-model reduction is 269ft/s -> 86ft/s, which is an improvement of roughly 183ft/s from ignorance
  - Despite the higher model MAE, the numbers actually show that the model *is still improving its predictive accuracy over ignorance.* The higher number is simply because ignorance error was raised when we introduced a significantly larger dataset with a wider variance in data.
  - Additionally, residuals are still behaving:
    - No obvious heteroscedastic explosion
    - Still no systematic bias
-Conclusion: *The model is still improving at a meaningful rate* I am satisfied enough with these results to continue introducing the next categorical features to see if they can continue to improve accuracy.

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

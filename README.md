# Handloading Muzzle Velocity Predictor

Personal machine learning project focused on modeling firearm muzzle velocity using publicly available handloading data. The system leverages physically meaningful features such as cartridge type, bullet weight, propellant charge, burn rate characteristics, and barrel length to study how predictive behavior evolves as domain context is introduced.

Early model baselines were intentionally constrained to single-cartridge datasets in order to preserve interpretability and isolate internal ballistic effects. Subsequent iterations expand across multiple cartridges, introducing geometric and behavioral diversity and evaluating the model’s ability to generalize across physically distinct ballistic regimes.

Data is ethically sourced from publicly available manufacturer load data. Limited anecdotal data from firearm forums is included for exploratory analysis but does not materially influence model training due to the dominance of manufacturer-supplied datasets.

Models are implemented using scikit-learn. Initial baselines utilized ordinary least squares (OLS) regression, with current iterations incorporating regularized regression (Ridge) as feature dimensionality increases.

This project is non-commercial and intended for educational and research-oriented exploration. Detailed model evolution, diagnostics, and ongoing progress notes are documented within the baseline subdirectory.
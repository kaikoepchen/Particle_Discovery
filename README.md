# Particle Discovery — B_s → μμ

search for the rare decay B_s → μμ in 10k signal + 10k background events. build a classifier, optimise the selection, figure out how long the experiment needs to run for a 5σ discovery.

## results

### core

BDT selection applied to all features (excluding MASS to keep the fit unbiased):

![ROC comparison](plots/rectangular_cuts.png)

AdaBoost on 7 features reaches 92.87% accuracy, 93.05% signal efficiency, 7.26% background efficiency. Feeding that working point into the composite-PDF toy MC gives median 1-year significance ≈ 10σ and a minimum experiment duration of ~0.5 yr for a 95% chance of a 5σ discovery.

![discovery duration](plots/discovery_duration.png)

### extensions

Each extension is self-contained — it re-runs the same background fit + toy MC as `mass_fit.ipynb` at its own working point, so the core notebook never depends on extension state.

Punzi FOM — pick the threshold that actually minimises experiment runtime instead of just maximising accuracy:

![Punzi FOM](plots/punzi_fom.png)

At the Punzi-optimal working point (BDT score ≥ 0.626): signal efficiency 59.0%, background efficiency 0.72%. ~10× stronger background rejection buys a sharper 1-year significance (~10.8σ) and the same T95 ≈ 0.5 yr with more headroom.

![discovery vs duration — Punzi](plots/discovery_duration_punzi.png)

A gradient-boosted classifier with 10-fold cross-validated grid search over `max_depth` and `learning_rate` picks `{max_depth: 5, learning_rate: 0.2}` with 98.2% CV accuracy. At its Punzi-optimal threshold (BDT score ≥ 0.970) it gives signal efficiency 87.4%, background efficiency 0.02%, median 1-year significance ~15.5σ, and T95 ~0.3 yr:

![CV tuning](plots/cv_tuning.png)
![Improved Punzi](plots/punzi_improved.png)
![discovery vs duration — improved](plots/discovery_duration_improved.png)

Wilks' theorem validity — check that q ~ chi2(1) holds at our sample size:

![Wilks validity](plots/wilks_validity.png)

## how to run

run the notebooks in order:

1. `features.ipynb` — rank features by Fisher score
2. `selection.ipynb` — rectangular cuts on top 3 features
3. `bdt.ipynb` — train AdaBoost, apply selection → writes `bdt_results.json`
4. `mass_fit.ipynb` — background fit, toy MC, discovery duration using the core BDT working point

extensions (independent — each reads the core BDT model / data and quotes its own discovery time):

5. `extensions/punzi_fom.ipynb` — Punzi threshold scan on the AdaBoost scores → writes `punzi_results.json`
6. `extensions/classifier_improvements.ipynb` — gradient boosting with 10-fold CV grid-search hyperparameter tuning + Punzi scan → writes `improved_results.json`
7. `extensions/wilks_validation.ipynb` — Wilks' theorem validity check with H0 toys

## files

```
data/                           signal + background samples
plots/                          all output figures
features.ipynb                  feature ranking
selection.ipynb                 rectangular cut optimisation
bdt.ipynb                       AdaBoost training + evaluation
mass_fit.ipynb                  significance + discovery duration (core WP)
extensions/                     punzi fom, classifier improvements, wilks validation
fisher_scores.csv               feature ranking (from features.ipynb)
cut_params.json                 optimal cuts (from selection.ipynb)
bdt_results.json                core BDT efficiencies (from bdt.ipynb)
bdt_model.pkl                   trained AdaBoost model
punzi_results.json              Punzi WP + extension-quoted T95
improved_model.pkl              trained gradient-boosted model
improved_results.json           CV + improved-Punzi WP + extension-quoted T95
fit_params.json                 background slope (from mass_fit.ipynb)
```

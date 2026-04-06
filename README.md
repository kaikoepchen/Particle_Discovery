## how to run

Run the notebooks in order — each one saves intermediate results that the next one loads.

1. `features.ipynb` — look at the data, rank features by Fisher score
2. `selection.ipynb` — find the best rectangular cut using the top 3 features
3. `bdt.ipynb` — train BDTs (3 features, then all 7), apply the selection
4. `mass_fit.ipynb` — fit the background shape, run toy MC, determine how long the experiment needs to run

The extension notebooks in `extensions/` can be run independently after the main pipeline:

5. `extensions/punzi_fom.ipynb` — Punzi figure of merit for threshold optimisation
6. `extensions/wilks_validation.ipynb` — validate Wilks' theorem with background-only toys
7. `extensions/classifier_improvements.ipynb` — feature engineering, k-fold CV, gradient boosting

---

## folder structure

```
data/
    signal_Bs2MuMu.txt
    background_combinatorial.txt

plots/

features.ipynb
selection.ipynb
bdt.ipynb
mass_fit.ipynb
fisher_scores.csv               saved by features.ipynb, loaded by selection + bdt
cut_params.json                 saved by selection.ipynb, loaded by bdt
bdt_results.json                saved by bdt.ipynb, loaded by mass_fit + extensions
bdt_model.pkl                   saved by bdt.ipynb (sklearn model)
fit_params.json                 saved by mass_fit.ipynb (background slope lambda)
improved_model.pkl              saved by classifier_improvements (gradient boosting model)
improved_results.json           saved by classifier_improvements (CV scores, efficiencies)

extensions/
    punzi_fom.ipynb             Punzi FOM threshold scan
    wilks_validation.ipynb      Wilks' theorem validity check
    classifier_improvements.ipynb   feature engineering + gradient boosting + k-fold CV

README.md
```

---

## what each notebook does

**features.ipynb**
Loads signal and background, plots a histogram for each of the 8 features, then computes the Fisher score for each one to rank their discriminating power. MASS is excluded from the BDT training later because we need the mass distribution unbiased for the fit. Saves the ranking to `fisher_scores.csv`.

**selection.ipynb**
Takes the top 3 non-MASS features from the Fisher ranking and searches for the rectangular cut combination (feature, direction, threshold) that maximises classification accuracy. Does a coarse grid search over percentile thresholds, then refines around the best found value. Saves the optimal cuts to `cut_params.json`.

**bdt.ipynb**
Trains a BDT using the same top 3 features and compares to the rectangular cut accuracy. Then adds all 7 non-MASS features and checks how much accuracy improves. Finally applies the 7-feature BDT to the full dataset and counts how many signal and background events pass. Saves the model and efficiencies to `bdt_results.json`.

**mass_fit.ipynb**
Fits the background mass distribution to a normalised exponential on [4, 6] GeV to get the slope. Then generates 1000 toy MC datasets for one year of running (50 signal x efficiency + 2000 background x efficiency events, with Poisson fluctuations). Fits each toy with a composite signal+background PDF and recovers the signal fraction. Computes the significance per toy using Wilks' theorem (test statistic q = 2 x delta NLL, significance = sqrt(q)). Finally scans over experiment durations to find the minimum runtime needed for a 95% chance of a >5 sigma discovery.

---

## extensions

**extensions/punzi_fom.ipynb**
The baseline optimises the BDT threshold for accuracy, which treats all misclassifications equally. The Punzi figure of merit replaces accuracy with a metric that directly measures discovery sensitivity: FOM = signal_efficiency / (n_sigma/2 + sqrt(B)). Scans the BDT output score, compares the Punzi-optimal and accuracy-optimal working points, and computes the resulting change in required experiment duration.

**extensions/wilks_validation.ipynb**
The significance calculation in mass_fit uses Wilks' theorem (q ~ chi2(1) under H0). But the signal fraction fs = 0 lies on the boundary of the parameter space, where the standard theorem does not strictly hold. Generates 500 background-only toy datasets, fits each, and compares the q distribution to chi2(1) to verify the asymptotic approximation is valid at our sample size.

**extensions/classifier_improvements.ipynb**
Improves the classifier itself in three ways. First, constructs 6 physically motivated features from the decay kinematics (momentum asymmetries, log-transforms, interaction terms). Second, evaluates all configurations with 10-fold stratified cross-validation instead of a single train/test split. Third, replaces AdaBoost with histogram-based gradient boosting (HistGradientBoostingClassifier). The best model (gradient boosting + 13 features) achieves 97.3% CV accuracy with 99.3% signal efficiency and 3.5% background efficiency, a 1.55x sensitivity gain over the baseline.


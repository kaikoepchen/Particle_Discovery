## how to run

Run the notebooks in order — each one saves intermediate results that the next one loads.

1. `features.ipynb` — look at the data, rank features by Fisher score
2. `selection.ipynb` — find the best rectangular cut using the top 3 features
3. `bdt.ipynb` — train BDTs (3 featur
es, then all 7), apply the selection
4. `mass_fit.ipynb` — fit the background shape, run toy MC, determine how long the experiment needs to run

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
bdt_results.json                saved by bdt.ipynb, loaded by mass_fit + punzi_fom
bdt_model.pkl                   saved by bdt.ipynb (sklearn model)
fit_params.json                 saved by mass_fit.ipynb (background slope lambda)

ProjectV_Discovery (3).pdf      
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
Fits the background mass distribution to a normalised exponential on [4, 6] GeV to get the slope. Then generates 1000 toy MC datasets for one year of running (50 signal × efficiency + 2000 background × efficiency events, with Poisson fluctuations). Fits each toy with a composite signal+background PDF and recovers the signal fraction. Computes the significance per toy using Wilks' theorem (test statistic q = 2 × delta NLL, significance = sqrt(q)). Finally scans over experiment durations to find the minimum runtime needed for a 95% chance of a >5 sigma discovery.



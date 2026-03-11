# Kinase Inhibitor Bioactivity Prediction

> Comparative Machine Learning for Kinase Inhibitor Bioactivity Prediction

A systematic, reproducible benchmark of four ML model families: random forest, SVR, multilayer perceptrons, and a residual network, for predicting small-molecule inhibitory potency (pIC50) against human kinase targets directly from molecular structure.

---


All models evaluated on a **Bemis–Murcko scaffold-based train/test split** (80/20), ensuring structurally novel compounds in the test set.

---

## Project Overview

Protein kinases are the single most targeted enzyme class in oncology. Experimental IC50 measurement is slow and expensive; QSAR modelling enables rapid computational pre-screening of virtual compound libraries before synthesis. This project:

1. Retrieves bioactivity data from **ChEMBL** for **70 human kinase targets**
2. Constructs a unified **1,102-feature** molecular representation combining:
   - ECFP4 Morgan fingerprints (radius=2, 2048 bits)
   - MACCS structural keys (166 public keys, 167-dim. vector)
   - Lipinski + extended physicochemical descriptors (12 features)
3. Applies variance thresholding and Pearson correlation filtering for feature selection
4. Benchmarks **7 model configurations** under identical conditions
5. Demonstrates that SVR with RBF kernel achieves the best generalisation at this dataset scale

---

## Repository Structure

```
bioactivity_prediction/
│
├── 1_ Data_Preprocessing.ipynb      # Data retrieval, cleaning, featurisation, scaffold split
├── 3_Random_Forest.ipynb            # Random Forest with GridSearchCV hyperparameter tuning
├── 4_Suport_Vector_Machine.ipynb    # SVR with RBF kernel, GridSearchCV tuning
├── 5_Neural_Network.ipynb           # MLP (baseline and regularised) architectures
├── 6_resNetwork.ipynb                 # Residual network with skip connections
├── Results.ipynb                  # Aggregated results, plots, cross-model comparison
│
├── Bioactivity_data.csv             # Raw ChEMBL bioactivity data
├── train_data.csv                   # Scaffold-split training set (13,911 compounds)
├── test_data.csv                    # Scaffold-split test set (3,333 compounds)
├── model_info.txt                   # Logged RMSE / R² for all model runs

```

---

## Methodology

### Dataset

- **Source:** ChEMBL (IC50 measurements, `standard_type = "IC50"`)
- **Targets:** 70 human kinase enzymes
- **Preprocessing:** desalting, deduplication, pIC50 conversion, outlier removal (MW > 1,000 Da, TPSA > 200 Å², RotBonds > 20)
- **Activity labels:** Active (IC50 ≤ 1,000 nM) / Inactive (IC50 ≥ 10,000 nM); intermediates removed
- **pIC50 range:** [3.00, 11.30] after noise filtering

$$\text{pIC50} = -\log_{10}\!\left(\text{IC50} \times 10^{-9}\right)$$

### Molecular Representation

| Feature Type | Dimension |
|---|:---:|
| Lipinski descriptors (MW, LogP, HBD, HBA) | 4 |
| Extended descriptors (TPSA, RotBonds, AromaticRings, …) | 8 |
| MACCS structural keys (166 public keys, 167-dim. vector) | 167 |
| Morgan fingerprints / ECFP4 (radius=2, 2048 bits) | 2,048 |
| **After variance threshold + correlation filter** | **1,102** |

### Train / Test Split

Bemis–Murcko scaffolds computed with RDKit; `GroupShuffleSplit(test_size=0.2, random_state=42)` assigns all compounds sharing a scaffold exclusively to train or test.

| Partition | Compounds | Unique Scaffolds | pIC50 Mean ± SD |
|---|:---:|:---:|:---:|
| Train | 13,911 (80.7%) | 5,851 | 6.59 ± 1.51 |
| Test | 3,333 (19.3%) | 1,463 | 6.49 ± 1.55 |

### Models

| Model | Key Hyperparameters |
|---|---|
| Random Forest | n_estimators, max_depth, max_features, max_samples tuned via GridSearchCV |
| SVR (RBF) | C ∈ {0.1,1,10,100}, ε ∈ {0.01,0.1,0.5,1.0}, γ ∈ {scale,0.01,0.1,1} |
| MLP Baseline | 512→256→128→64, ReLU, BatchNorm, Dropout(0.3,0.2), Adam |
| MLP Regularised | Higher Dropout (0.4→0.3→0.2), ReduceLROnPlateau |
| ResNet | 2 residual blocks (512→256, 256→128), skip projections, L2(1e-4) |

---

## Key Findings

- **SVR > RF > MLP > ResNet** — consistent with MoleculeNet benchmarks at ~14k training samples
- **Random Forest** suffers from systematic prediction clipping on novel scaffolds with extreme pIC50 values 
- **SVR** is not bounded by training target extremes; the continuous RBF kernel function can extrapolate, producing a more symmetric residual distribution
- **Deep learning underperforms** classical methods here — the ~13,900-sample dataset cannot saturate the parameter space of overparameterised architectures
- **Scaffold splitting** produces substantially more conservative R² estimates than random splitting (≈ 0.15–0.25 lower), better reflecting prospective deployment

---

## Setup & Usage

### Prerequisites

```bash
pip install rdkit scikit-learn tensorflow pandas numpy matplotlib seaborn
```

Or with conda:

```bash
conda install -c conda-forge rdkit
pip install scikit-learn tensorflow pandas numpy matplotlib seaborn
```

### Run Order

Execute the notebooks in order:

1. **`1_ Data_Preprocessing.ipynb`** — cleans raw data, computes fingerprints, applies feature selection, performs scaffold split, saves `train_data.csv` / `test_data.csv`
2. **`3_Random_Forest.ipynb`** — trains and evaluates Random Forest
3. **`4_Suport_Vector_Machine.ipynb`** — trains and evaluates SVR
4. **`5_Neural_Network.ipynb`** — trains MLP (baseline and regularised)
5. **`6_resNetwork.ipynb`** — trains residual network
6. **`Results.ipynb`** — aggregates results, generates comparison plots

> All notebooks use `random_state = 42` and fixed seeds (`np.random.seed`, `tf.random.set_seed`, `PYTHONHASHSEED`) for full reproducibility.

---

## Data Source

Bioactivity data retrieved from **ChEMBL** (open access, EMBL-EBI):
https://www.ebi.ac.uk/chembl/

Molecular descriptors and fingerprints computed with **RDKit**:
https://www.rdkit.org


---

## License

Data from ChEMBL is provided under the [Creative Commons Attribution-ShareAlike 3.0 Unported License](https://creativecommons.org/licenses/by-sa/3.0/).

---



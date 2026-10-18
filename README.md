# EGFR Inhibitor Potency Prediction (QSAR)

A Quantitative Structure-Activity Relationship (QSAR) machine learning pipeline that predicts the potency (pIC50) of chemical compounds against the Epidermal Growth Factor Receptor (EGFR), a well-validated cancer drug target, directly from molecular structure.

## Overview

Early-stage drug discovery involves screening thousands of candidate compounds. A model that can estimate potency from structure alone helps prioritize which compounds are worth synthesizing and testing. This project builds an end-to-end pipeline, from raw ChEMBL data retrieval through featurization, modeling, and interpretation, to evaluate how well structural and physicochemical features predict EGFR inhibition.

## Results

| Metric | Test Set |
|---|---|
| R² | 0.71 |
| RMSE | [X] pIC50 units |

- The model explains roughly 71% of the variance in pIC50 on held-out compounds.
- Because pIC50 is a log10 scale, an RMSE under 1.0 means typical predictions fall within a 10-fold error of true potency.
- A single Morgan fingerprint bit (`FP_343`) accounted for roughly half of total feature importance, well above any physicochemical descriptor (`MolWt` ranked next). See the notebook for the substructure this bit encodes.

## Pipeline

1. **Data ingestion**: Query the ChEMBL API for human EGFR (UniProt P00533), filtered to binding assays reporting IC50 in nM. Records are streamed in batches of 1,000 with a 5,000-record cap.
2. **Cleaning**: Drop records with missing SMILES or values, convert IC50 to pIC50 (`9 - log10(IC50 nM)`), remove duplicate SMILES, and remove invalid SMILES.
3. **EDA**: Inspect the pIC50 distribution (histogram and boxplot) and assess outliers. All outliers were on the high-potency end and were retained as likely genuine high-affinity inhibitors.
4. **Featurization**:
   - RDKit molecular descriptors: molecular weight, LogP, H-bond donors/acceptors, TPSA, rotatable bonds, aromatic ring count
   - Morgan fingerprints: 1,024-bit vectors, radius 2
5. **Train/test split**: 80/20 split with a fixed random seed for reproducibility.
6. **Modeling**: Random Forest regressor (no feature scaling required for mixed continuous and binary features).
7. **Evaluation**: R², RMSE, predicted-vs-actual plot, and feature importances. The top fingerprint bit is traced back to its substructure for interpretability.

## Installation

```bash
git clone [repo-url]
cd cancer-drug-predictor
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Core dependencies: `chembl_webresource_client`, `rdkit`, `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `jupyter`.

## Usage

Open and run the notebook top to bottom:

```bash
jupyter notebook [notebook-name].ipynb
```

Note: the ChEMBL download requires an internet connection and may take a few minutes. ChEMBL is updated regularly, so results may vary slightly depending on when the data is pulled.

## Project Structure

```
cancer-drug-predictor/
├── [notebook-name].ipynb    # Full pipeline: ingestion, EDA, featurization, modeling, evaluation
├── requirements.txt
└── README.md
```

## Limitations and Future Work

- **Single train/test split**: Results come from one random split. K-fold cross-validation (reporting Q²) would give a more reliable generalization estimate on a dataset of this size.
- **Random split vs. scaffold split**: Structurally similar compounds can land in both train and test sets, which may overestimate performance on novel chemotypes. A scaffold-based split is a more stringent test.
- **No hyperparameter tuning**: The Random Forest uses near-default settings. Tuning, and comparison against gradient boosting, are natural next steps.
- **Assay heterogeneity**: Data is pooled across labs and assays. Filtering to binding IC50 reduces but does not eliminate inter-assay variability.
- **Classification reframing**: Binning compounds as active vs. inactive (e.g., pIC50 above a threshold) would mirror how screening decisions are often made in practice, evaluated with ROC-AUC or PR-AUC.

## Data Source

[ChEMBL](https://www.ebi.ac.uk/chembl/), maintained by EMBL-EBI.

## Author

Zoe, [LinkedIn / GitHub / email]
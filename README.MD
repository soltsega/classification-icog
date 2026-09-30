# iCog Week 1 Training Project

## Enzyme function classification

This repository contains the iCog Week 1 machine learning training project. The goal is to predict the top-level enzyme commission (EC) class, 1 through 6, from protein amino-acid sequences using hand-crafted sequence features and ensemble learning.

**Task mentor:** Kaletsidk Ayalew, who assigned the project.

## Project contents

```text
classification-icog/
├── README.MD
├── requirements.txt
├── instruction.ipynb
├── Enzyme_Function_Classification_Ensemble_Colab.ipynb
├── notebooks/
│   └── Enzyme_Function_Classification_Ensemble_Colab.ipynb
└── artifacts/                 # Generated model and evaluation outputs
```

The project brief and grading requirements are in `instruction.ipynb`. The modeling notebook is `Enzyme_Function_Classification_Ensemble_Colab.ipynb`; the same notebook is mirrored under `notebooks/` for notebook-focused workflows; keep the two copies in sync when editing.

## Set up locally

Use Python 3.10 or newer. From the repository root, create and activate a virtual environment, then install the dependencies:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Start Jupyter from the repository root so generated files use the repository's `artifacts/` directory:

```powershell
jupyter notebook Enzyme_Function_Classification_Ensemble_Colab.ipynb
```

The dataset is **DanielHesslow/SwissProt-EC** on Hugging Face. The first run needs internet access to stream a bounded sample and may take time to download/cache the records. The notebook caps the cleaned modeling dataset at 10,000 rows (`MAX_ROWS = 10000`) and tunes on a 2,500-row training subset with two parameter draws per model and five-fold CV. This keeps the required comparisons while reducing runtime. Google Drive mounting and Colab-only setup are not needed for local use.

## What the notebook does

- Extracts the first digit of the EC annotation and cleans protein sequences.
- Creates a stratified 70% training, 15% validation, and 15% test split.
- Builds amino-acid composition, dipeptide, and selected tripeptide features.
- Compares Random Forest, LightGBM, and SVM with five-fold cross-validation.
- Evaluates hard voting, soft voting, and stacking ensembles.
- Reports Macro F1, MCC, AUPRC, accuracy, per-class scores, confusion matrix, ROC curves, and error analysis.

Optional SVD embeddings and SHAP explanations can be enabled in the notebook, with additional runtime and memory cost.

## Outputs

The notebook saves generated artifacts under `artifacts/`, including the selected model and its feature transforms, plus test metrics. These files are created when the notebook completes successfully. The `artifacts/` directory is intentionally not committed.


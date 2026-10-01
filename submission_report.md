# Enzyme Function Classification Using Sequence Features and Ensemble Learning

## 1. Introduction

Enzyme function is represented by a hierarchical EC classification. This project predicts only the first EC digit, grouping proteins into six broad enzyme classes: oxidoreductases (EC 1), transferases (EC 2), hydrolases (EC 3), lyases (EC 4), isomerases (EC 5), and ligases (EC 6). Predicting this broad category from sequence alone is a challenging task because the representations used here summarize amino-acid composition and short local sequence patterns rather than full three-dimensional structure or biochemical context.

## 2. Dataset

The data came from the `DanielHesslow/SwissProt-EC` dataset hosted on Hugging Face. A bounded streaming sample was loaded, and the first valid EC digit from 1 through 6 was used as the target. The recorded run contained 5,210 rows after label extraction and sequence cleaning. Sequences shorter than five residues, missing sequences, and sequences containing non-canonical amino-acid symbols were excluded. Exact duplicate sequences were removed before splitting to reduce the risk of identical sequences appearing in both training and test sets.

The cleaned data was split into training, validation, and test sets using a stratified 70/15/15 split with random state 42. The test set contained 782 proteins. Stratification preserved approximate class proportions across the splits. The notebook used a streamed subset of the source dataset, so the results describe this sampled dataset and split rather than the full SwissProt-EC collection.

## 3. Methodology

The task was treated as six-class supervised classification. Data was split into training, validation, and test partitions using stratification and a 70/15/15 ratio. All learned feature transformations were fitted on training data, and model/representation selection used validation Macro F1. The test set was reserved for final evaluation. Exact duplicate sequences were removed before splitting.

## 4. Feature Engineering

Three sequence representations were evaluated. AAC represents each sequence with 20 normalized amino-acid frequencies. Dipeptide composition represents adjacent amino-acid pairs, with up to 400 possible pair features. Tripeptide composition represents consecutive groups of three residues; the training-fitted feature selector retained 100 features for modeling. The optional TruncatedSVD representation was disabled in this run.

## 5. Models

Random Forest, LightGBM, and a linear SVM were tuned using stratified five-fold cross-validation and Macro F1. Tuning used a stratified subset of up to 800 training examples for the primary comparisons and up to 600 for the dipeptide tuning track; selected configurations were then refit on the full training split. Macro F1 was the primary model-selection metric because it weights each class equally. Accuracy, Matthews correlation coefficient (MCC), macro one-vs-rest ROC-AUC, and macro one-vs-rest average precision (AUPRC) were also used to assess performance. The final model was selected using validation Macro F1, not test performance.

## 6. Ensemble Methods

The notebook defines hard voting, soft voting, and stacking. The stacking design uses logistic regression as its meta-classifier. In the saved test comparison, only hard voting had recorded results; soft-voting and stacking results were not available, so they are not interpreted here.

## 7. Results

### Representation comparison

The strongest validation Macro F1 for each representation was:

| Representation | Best validation Macro F1 |
|---|---:|
| Dipeptide | 0.489 |
| AAC | 0.392 |
| Selected tripeptide | 0.384 |

Dipeptide composition was selected for the final model comparison. The LightGBM model was the strongest recorded model for that representation.

### Held-out test performance

| Model | Macro F1 | Accuracy | MCC | Macro AUPRC |
|---|---:|---:|---:|---:|
| LightGBM | 0.495 | 0.546 | 0.396 | 0.535 |
| Hard voting | 0.327 | 0.400 | 0.260 | 0.368 |
| Random Forest | 0.312 | 0.445 | 0.219 | 0.411 |
| SVM | 0.039 | 0.134 | 0.000 | 0.259 |

The LightGBM model outperformed the other models in the saved comparison. Hard voting did not improve on the strongest individual model.

The selected LightGBM model had a macro one-vs-rest ROC-AUC of approximately 0.798. Class-level ROC-AUC ranged from 0.737 for EC 5 to 0.898 for EC 6. The macro one-vs-rest AUPRC was approximately 0.535.

### Per-class performance

| EC class | Precision | Recall | F1 | Test support |
|---|---:|---:|---:|---:|
| EC 1 | 0.495 | 0.429 | 0.459 | 105 |
| EC 2 | 0.569 | 0.639 | 0.602 | 285 |
| EC 3 | 0.527 | 0.560 | 0.543 | 191 |
| EC 4 | 0.543 | 0.373 | 0.442 | 67 |
| EC 5 | 0.588 | 0.227 | 0.328 | 44 |
| EC 6 | 0.552 | 0.644 | 0.595 | 90 |

EC 2 had the highest observed class-level F1 (0.602). EC 5 had the lowest F1 (0.328), mainly reflecting its low recall (0.227). The most frequent off-diagonal errors were EC 3 predicted as EC 2 (58 cases) and EC 2 predicted as EC 3 (53 cases). These results indicate that the model had difficulty separating these two classes in this sample.

### Feature interpretation

The notebook's SHAP analysis was run on a sample of up to 100 held-out test examples. The largest mean absolute SHAP contributions included the dipeptide features `LR`, `PR`, `WG`, `LL`, and `DG`. These are predictive associations in the fitted model; they do not establish causal biochemical mechanisms.

The recorded results support dipeptide composition as the most effective evaluated representation for this run. Adding local order information beyond overall amino-acid frequencies appears useful. However, the test Macro F1 of 0.495 indicates substantial remaining classification error. EC 5 was frequently missed, and the largest confusions were between EC 2 and EC 3. Class supports were unequal: EC 2 had 285 test examples and EC 5 had 44, so estimates for smaller classes are less stable. SHAP analysis on up to 100 held-out examples ranked `LR`, `PR`, `WG`, `LL`, and `DG` among the largest mean absolute contributions. These are predictive associations, not causal biochemical mechanisms.

## 8. Limitations

The notebook configuration sets `MAX_ROWS` to 1,500, but the recorded cleaned dataset contains 5,210 rows. The loader takes up to four times `MAX_ROWS` before cleaning, and the current code does not apply a final 1,500-row cap to the cleaned dataframe. Therefore, the reported results correspond to the 5,210-row run, not to a 1,500-row modeling dataset.

The notebook configuration sets `MAX_ROWS` to 1,500, but the recorded cleaned dataset contains 5,210 rows. The loader takes up to four times `MAX_ROWS` before cleaning, and the current code does not apply a final 1,500-row cap to the cleaned dataframe. Therefore, the reported results correspond to the 5,210-row run, not to a 1,500-row modeling dataset. The saved comparison does not include fitted soft-voting or stacking test results; these required ensemble methods need to be rerun before drawing conclusions about them.

The notebook also contains narrative text that refers to Google Drive and 128 SVD components, while the recorded workflow streams from Hugging Face and configures 64 SVD components. SVD was disabled, so it did not affect these results. Exact duplicate removal does not ensure that homologous proteins are separated across splits; a random split may overstate generalization to evolutionarily distant proteins. A clean restart and run-all should be performed before submission to confirm that saved outputs match the current notebook code and configuration.

## 9. Conclusion

For the recorded run, a LightGBM classifier trained on dipeptide composition produced the best held-out results, with Macro F1 of 0.495 and accuracy of 0.546. It outperformed the recorded hard-voting ensemble and the other individual models. EC 5 was the hardest class by F1, and errors were concentrated in confusion between EC 2 and EC 3. These findings are limited to the sampled dataset and random split. A reproducibility run with the intended row limit, verified ensemble fits, and preferably a homology-aware split would provide a stronger basis for assessing generalization.

## 10. References

1. DanielHesslow, `SwissProt-EC` dataset, Hugging Face.
2. Scikit-learn documentation for feature extraction, model selection, classification metrics, and ensemble estimators.
3. LightGBM documentation for gradient-boosted decision trees.
4. SHAP documentation for tree-based model explanations.
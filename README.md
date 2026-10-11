# LineSense: Context-Aware Line-Level Software Defect Prediction

A deep learning project for software defect prediction and suspicious source-code line ranking using CodeBERT, a Preceding Line-Aware (PLA) module, and Dual-Attention Inter-Line Information Fusion (DAIF).

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [Technologies Used](#technologies-used)
- [Dataset](#dataset)
- [Model Architecture](#model-architecture)
- [Methodology](#methodology)
- [Experimental Setup](#experimental-setup)
- [Results](#results)
- [Evaluation Metrics](#evaluation-metrics)
- [Limitations](#limitations)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Reproducibility](#reproducibility)
- [Future Improvements](#future-improvements)
- [References](#references)

## Overview

LineSense is a machine learning project that investigates context-aware software defect prediction and line-level defect localization. The goal is to help developers identify potentially defective source-code files and prioritize suspicious lines for code review, testing, debugging, and maintenance.

The implementation uses CodeBERT to generate source-code line representations, PLA to incorporate preceding-line context, and DAIF to model relationships between lines through attention mechanisms.

The project evaluates both Within-Project Defect Prediction (WPDP) and Cross-Project Defect Prediction (CPDP) using nine Java open-source projects.

## Problem Statement

Software defects can increase maintenance costs, reduce reliability, and make debugging more time-consuming. Reviewing every source-code line is often impractical for large software systems.

Software defect prediction aims to identify files that are likely to contain defects. Line-level defect localization goes further by ranking individual source-code lines according to their suspiciousness.

This project explores whether contextual code representations and attention-based information fusion can support file-level prediction and line-level suspiciousness ranking.

## Objectives

- Generate contextual representations of source-code lines using CodeBERT.
- Incorporate preceding-line information through the PLA module.
- Model inter-line relationships using the DAIF module.
- Evaluate within-project and cross-project defect prediction.
- Measure file-level classification and line-level ranking performance.
- Study the effects of architectural components through ablation experiments.
- Identify limitations and opportunities for improving line-level localization.

## Technologies Used

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| PyTorch | Deep learning model implementation and training |
| CodeBERT | Source-code representation |
| Hugging Face Transformers | Loading the pretrained encoder |
| Scikit-learn | Evaluation metrics and supporting utilities |
| Pandas | Tabular data processing |
| NumPy | Numerical operations |
| Kaggle Notebooks | Experiment execution and GPU access |
| Git and GitHub | Version control and project documentation |

## Dataset

The experiments use a benchmark comprising nine Java open-source projects and multiple software releases.

The projects are:

- ActiveMQ
- Camel
- Derby
- Groovy
- HBase
- Hive
- JRuby
- Lucene
- Wicket

The benchmark documented in the project contains 32 releases, 73,384 files, and approximately 11.8 million lines of code.

The experimental protocol includes:

- **WPDP:** Training and evaluating within the same project using version-based splits.
- **CPDP:** Evaluating a model trained on one project against other projects without retraining.

Dataset availability, licensing, and preprocessing requirements should be checked before redistributing any dataset files.

## Model Architecture

The LineSense pipeline consists of the following main components.

### 1. CodeBERT Encoder

CodeBERT generates contextual vector representations of source-code lines. The implementation uses the pretrained `microsoft/codebert-base` model as a frozen feature extractor.

The encoder produces 768-dimensional representations that are passed to the downstream modules.

### 2. Preceding Line-Aware (PLA) Module

The PLA module incorporates preceding-line context into each line representation. Its recurrent, matrix-memory-style design allows the representation of a line to retain information from earlier lines in the file.

### 3. Dual-Attention Inter-Line Information Fusion (DAIF)

DAIF models relationships between line representations through self-attention and cross-attention operations. This component is intended to capture dependencies between different parts of a source file.

### 4. Prediction and Ranking

The downstream components support file-level defect prediction and line-level suspiciousness ranking.

File-level predictions estimate whether a file is defective. Line-level scores are used to prioritize lines for inspection.

The architecture is based on the PLEASE research framework; this repository documents the LineSense implementation and its experimental evaluation rather than claiming the underlying architecture as an entirely new invention.

## Methodology

The main experimental workflow is:

1. Load the project data and prepare source-code lines.
2. Clean source-code text according to the preprocessing procedure.
3. Generate and cache CodeBERT line embeddings.
4. Process line sequences through the PLA module.
5. Apply DAIF to model inter-line relationships.
6. Generate file-level predictions and line-level scores.
7. Train the downstream modules using the configured loss and optimization procedure.
8. Select checkpoints using validation performance.
9. Evaluate the trained models on held-out project versions.
10. Conduct cross-project evaluations and ablation experiments.

## Experimental Setup

The documented configuration includes:

- **Framework:** PyTorch and Hugging Face Transformers
- **Encoder:** Frozen CodeBERT-base
- **PLA hidden dimension:** 256
- **DAIF hidden dimension:** 128
- **Optimizer:** Adam
- **Learning rate:** 0.0001
- **Training budget:** 10 epochs per project
- **Random seed:** 42
- **Loss:** Weighted binary cross-entropy

The experiments were conducted in a Kaggle GPU environment. Actual hardware availability, runtime, cached embeddings, and checkpoint requirements may affect reproducibility.

## Results

The following values are the overall results reported in the project thesis. They should be interpreted as results from this implementation, not as guaranteed performance on new datasets.

### Within-Project Defect Prediction (WPDP)

| Metric | Reported result |
|---|---:|
| Average AUC | 0.772 |
| Balanced Accuracy | 0.597 |
| Matthews Correlation Coefficient | 0.103 |
| Recall@Top20%LOC | 0.117 |
| Effort@Top20%Recall | 0.429 |

### Cross-Project Defect Prediction (CPDP)

| Metric | Reported result |
|---|---:|
| Average AUC | 0.740 |
| Balanced Accuracy | 0.539 |
| Matthews Correlation Coefficient | 0.056 |
| Recall@Top20%LOC | 0.143 |
| Effort@Top20%Recall | 0.4 |

The CPDP results summarize 256 source-target version evaluations, as documented in the thesis.

### Results Interpretation

The average AUC values indicate that the model can rank defective and non-defective files better than random ordering in the evaluated settings.

However, Balanced Accuracy and MCC remain relatively low in some projects, indicating classification-threshold and class-imbalance challenges.

The WPDP Recall@Top20%LOC result of 0.117 also indicates that line-level localization remains a significant limitation. File-level ranking performance should not be interpreted as proof of equally strong line-level localization.

## Evaluation Metrics

- **AUC:** Measures how well the model ranks defective files above non-defective files across classification thresholds.
- **Balanced Accuracy:** Averages sensitivity and specificity.
- **Matthews Correlation Coefficient (MCC):** Measures binary classification quality while accounting for all four confusion-matrix outcomes.
- **Recall@Top20%LOC:** Measures the fraction of defective lines identified within the top 20% of inspected lines of code.
- **Effort@Top20%Recall:** Measures the fraction of code-inspection effort required to identify 20% of defective lines.

## Limitations

- Line-level localization performance requires further improvement.
- Severe class imbalance can affect threshold-based classification metrics.
- Long source files can create high memory requirements for attention operations.
- Cached embeddings and trained checkpoints may be required to avoid repeating expensive computation.
- Results depend on the selected projects, releases, preprocessing, and evaluation protocol.
- Reproducing the full experiment may require a compatible GPU and substantial runtime.

## Repository Structure

```text
linesense-defect-prediction/
├── LineSenseProject.ipynb
├── README.md
└── .gitignore
```

Additional files, such as dependency lists, diagrams, or result exports, can be added as the project is organized for reproducibility.

## How to Run

### Option 1: Run in Kaggle

1. Open the notebook in this repository.
2. Download the notebook or import it into Kaggle.
3. Configure the required dataset inputs.
4. Enable a compatible GPU if required.
5. Install any missing dependencies.
6. Ensure cached embeddings and model checkpoints are available when needed.
7. Execute the cells in the required order.
8. Review the generated tables, metrics, and plots.

### Option 2: Run in Jupyter

1. Clone or download the repository.
2. Install Python and the dependencies required by the notebook.
3. Start Jupyter Notebook or JupyterLab.
4. Open `LineSenseProject.ipynb`.
5. Configure dataset paths and model inputs.
6. Run the notebook cells in order.

A fresh environment may require additional setup because the notebook can depend on external datasets, pretrained-model downloads, cached embeddings, or previously saved checkpoints.

## Reproducibility

For meaningful reproduction, use the same dataset versions, preprocessing steps, train-validation-test splits, model configuration, random seed, and evaluation protocol.

The notebook and its outputs should be inspected to determine which stages can run from scratch and which require existing cached data or checkpoints.

Do not treat saved notebook outputs alone as proof that every experiment can be reproduced in a fresh environment.

## Future Improvements

- Improve line-level defect localization and ranking.
- Investigate class-imbalance handling and threshold calibration.
- Evaluate alternative line-ranking strategies.
- Reduce memory usage for very long source files.
- Improve experiment tracking and checkpoint management.
- Add a dependency file and clearer dataset setup instructions.
- Include reproducible plots and per-project result summaries.

## References

The implementation is based on the PLEASE line-level software defect prediction research framework. Cite the original PLEASE paper and any additional datasets, libraries, and methods used in the final academic version of this project.

## Author

**Project:** LineSense — Context-Aware Line-Level Software Defect Prediction

**Repository:** [linesense-defect-prediction](https://github.com/yash4579/linesense-defect-prediction)

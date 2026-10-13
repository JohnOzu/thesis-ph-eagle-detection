# AGENTS.md

## Project Overview

This project develops a deep learning model for **binary classification of Philippine eagle bird calls**.

The model has two possible classification outputs:

1. **Eagle Call**: The target Philippine eagle call is detected.
2. **No Eagle Call**: The target eagle call is not detected.

The project focuses on audio-based bird-call classification and may involve audio preprocessing, feature extraction, spectrogram generation, deep learning model training, evaluation, and visualization.

Agents working on this project should prioritize reproducibility, clear experimentation, proper dataset separation, and organized outputs.

---

## Project Structure

The project follows a **modular, experiment-centric structure** where each experiment lives in its own self-contained directory under `experiments/`:

```text
project-root/
├── data/
│   ├── raw/
│   ├── preprocessed/
│   └── features/
│
├── experiments/
│   ├── dataset-exploration/
│   │   ├── dataset-exploration.ipynb
│   │   └── outputs/
│   │       └── figures/
│   │
│   ├── preprocess/
│   │   ├── preprocess.ipynb
│   │   └── outputs/
│   │       └── figures/
│   │
│   ├── feature-extraction/
│   │   ├── feature-extraction.ipynb
│   │   └── outputs/
│   │       ├── figures/
│   │       └── spectrograms/
│   │
│   ├── crnn-gru-training/
│   │   ├── crnn-gru-5s-training.ipynb
│   │   ├── crnn-gru-3s-training.ipynb
│   │   ├── models/
│   │   └── outputs/
│   │       ├── 3-seconds/
│   │       │   ├── <model>/<partition>/
│   │       │   ├── splits/
│   │       │   └── summary/
│   │       └── multiscale-comparison.csv
│   │
│   ├── birdnet-transfer-learning/
│   │   ├── birdnet-transfer-learning.ipynb
│   │   ├── models/
│   │   └── outputs/
│   │       ├── confusion-matrices/
│   │       ├── training-history/
│   │       ├── figures/
│   │       └── multiscale-comparison.csv
│   │
│   ├── mobilenetv3-training/
│   │   ├── mobilenetv3-5s-training.ipynb
│   │   ├── mobilenetv3-3s-training.ipynb
│   │   ├── models/
│   │   └── outputs/
│   │       ├── confusion-matrices/
│   │       ├── training-history/
│   │       ├── figures/
│   │       └── multiscale-comparison.csv
│   │
│   ├── species-error-analysis/
│   │   ├── species-error-analysis.ipynb
│   │   └── outputs/
│   │       ├── figures/
│   │       └── species-breakdown.csv
│   │
│   └── model-benchmark/
│       ├── model-benchmark.ipynb
│       └── outputs/
│           ├── figures/
│           └── master-benchmark.csv
│
├── src/
├── AGENTS.md
├── README.md
└── requirements.txt
```

---

### `data/`

Contains datasets used throughout the project.

#### `data/raw/`

Contains the original, unmodified audio datasets organized into target and negative corpora:

```text
data/raw/
├── eagle-audios/
│   ├── audios/            # Original recordings containing Philippine Eagle calls (.wav)
│   ├── selectionTables/   # Verified Raven selection tables for call annotations
│   └── raw-metadata.csv   # Catalog and recording site metadata for eagle audios
└── hard-negatives/
    ├── audios/            # Field recordings of sympatric non-eagle avian species (.wav, .mp3, .m4a)
    └── raw-metadata.csv   # Macaulay Library / eBird catalog metadata
```

Rules:

* Never modify or overwrite files in `data/raw/`.
* Treat raw data as immutable.
* Any preprocessing or transformation should produce files in another directory (such as `data/preprocessed/`).
* Do not commit large raw datasets unless explicitly required by the project.

#### `data/preprocessed/`

Contains sliced audio segments generated from raw recordings (e.g. 3.0-second and 5.0-second windows).

#### `data/features/`

Contains extracted features generated from audio.

Examples include:

* Mel spectrograms
* PCEN representations
* BirdNET foundation embeddings
* Preprocessed feature arrays

Feature generation must remain reproducible through experiment notebooks or scripts.

---

### `experiments/`

Contains self-contained experiment directories.

Every experiment folder:
1. Must use **semantic kebab-case without numerical ordering prefixes** (e.g. `dataset-exploration/`, `crnn-gru-training/`, `mobilenetv3-training/`).
2. Must contain the primary Jupyter notebook named identically to the experiment folder (or dedicated scale notebooks, e.g. `crnn-gru-5s-training.ipynb` and `crnn-gru-3s-training.ipynb`, for memory isolation).
3. Must contain an `outputs/` folder for all generated artifacts (figures, CSVs, reports).
4. If the experiment trains models, it must contain a local `models/` directory for its checkpoints and weights.

Do not create global, detached output or model folders. All artifacts must live inside their corresponding experiment directory.

---

## Experiment Output Conventions

All figures, plots, confusion matrices, and saved analysis tables must be stored inside the local `outputs/` folder of that specific experiment:

```text
experiments/<experiment-folder>/outputs/
├── figures/
├── confusion-matrices/
├── training-history/
├── spectrograms/
└── <name>.csv
```

### Per-Model Outputs

When a notebook trains several models on several partitions, group the outputs per model. The 3.0-second CRNN-GRU notebook uses this layout:

```text
experiments/crnn-gru-training/outputs/3-seconds/
├── <model>/                  # e.g. published-baseline, standard, cost-sensitive, edge-optimized
│   └── <partition>/          # recording-split, loso-davao, loso-cotabato, minor-sites
│       ├── confusion-matrix.png
│       ├── training-history.png
│       └── statistics.txt
├── splits/                   # split distribution and leak check figures, shared by all models
└── summary/                  # tables and figures that compare models
```

The folder names carry the model and the partition, so the file names inside stay short and are the same for every model.

### Clean Output Naming Convention

**Do not prepend the experiment folder name to output files.** 

Because files are already isolated inside their dedicated experiment directory, prefixes are redundant. Use clean, descriptive kebab-case:

Prefer:
```text
outputs/figures/eagle-geographic-distribution.png
outputs/figures/paradigm-comparison.png
outputs/confusion-matrices/3s-std-confusion-matrix.png
outputs/training-history/3s-std-training-history.png
outputs/multiscale-comparison.csv
```

Avoid:
```text
01-dataset-exploration-eagle-geographic-distribution.png
04-crnn-gru-model-training-3s-std-confusion-matrix.png
Figure1.png
confusion_matrix.png
output.png
```

---

## Model Artifact Conventions

When an experiment trains models, all artifacts must be saved inside that experiment's local `models/` directory:

```text
experiments/<experiment-folder>/models/
├── <scale>/
│   └── <split>/
│       ├── <model-name>.keras
│       ├── <model-name>-int8.tflite
│       ├── training_history.json
│       └── checkpoints/
│           └── best_model.keras
```

Examples:
```text
experiments/crnn-gru-training/models/3-seconds/standard/crnn-gru-std.keras
experiments/crnn-gru-training/models/3-seconds/standard/crnn-gru-std-int8.tflite
experiments/birdnet-transfer-learning/models/standard-split/birdnet-head-standard.keras
```

Do not overwrite existing trained models unless explicitly requested.

---

## Cross-Model Benchmarking & Comparative Analysis

Individual model training notebooks (`crnn-gru-training`, `birdnet-transfer-learning`, `mobilenetv3-training`) must remain **decoupled and self-contained**:
* A training notebook evaluates only its own models and exports its own performance table (e.g. `multiscale-comparison.csv`).
* Training notebooks must **not** load or depend on other notebooks' benchmark tables.

All cross-model benchmarking, multi-paradigm comparative plots, and overall project leaderboards belong exclusively in **dedicated comparative analysis notebooks** (e.g. `experiments/model-benchmark/`).

---

## Notebook Cell Conventions

Every executable notebook cell must have a descriptive title immediately before it.

Use Markdown cells as section titles formatted as Markdown headers (e.g. `## 1. Imports and Configs` or `### 1. Imports and Configs`).

The title should follow this format:

```markdown
## 1. Imports and Configuration
```

The numbering indicates the order or logical progression of the notebook.

For example:

```markdown
## 1. Imports and Configuration

[code cell]


## 2. Load Dataset

[code cell]


## 3. Inspect Dataset

[code cell]


## 4. Visualize Class Distribution

[code cell]


## 5. Train Model

[code cell]


## 6. Evaluate Model

[code cell]
```

Every executable notebook cell should have a corresponding title cell before it.

Titles should describe **what the following cell does**, rather than simply describing the code itself.

Prefer:

```markdown
## 5. Generate Mel Spectrogram
```

over:

```markdown
## 5. Run Code
```

---

## Notebook Reproducibility

Notebooks should be written so that another person can understand and reproduce the experiment.

Whenever practical:

* Set random seeds (`SEED = 42`).
* Define configuration values near the beginning of the notebook.
* Avoid hard-coded absolute paths.
* Use dynamic project-relative paths.
* Clearly identify input datasets.
* Clearly identify generated outputs.
* Record important model hyperparameters.
* Record training configuration.
* Record evaluation metrics.

---

## Path Handling

Avoid absolute paths such as:

```text
/home/user/project/data/raw/...
```

Use dynamic project root resolution so notebooks execute reliably from any directory depth:

```python
from pathlib import Path

def find_project_root():
    p = Path.cwd().resolve()
    while p != p.parent:
        if (p / "data").exists() and (p / "AGENTS.md").exists():
            return p
        p = p.parent
    return Path.cwd().resolve().parent.parent

PROJECT_ROOT = find_project_root()
DATA_DIR = PROJECT_ROOT / "data"
RAW_DATA_DIR = DATA_DIR / "raw"
FEATURES_DIR = DATA_DIR / "features"

EXPERIMENT_DIR = Path.cwd().resolve()
OUTPUTS_DIR = EXPERIMENT_DIR / "outputs"
FIGURES_DIR = OUTPUTS_DIR / "figures"
MODELS_DIR = EXPERIMENT_DIR / "models"
```

---

## Machine Learning Workflow

The general workflow follows:

```text
Raw Audio
    ↓
Data Exploration (01)
    ↓
Audio Preprocessing (02)
    ↓
Feature Extraction (03)
    ↓
Model Experiments (04, 05, 06)
    ↓
Species Error Analysis (07)
    ↓
Comparative Benchmarking (08)
```

---

## Binary Classification

The task is a binary classification problem:

```text
0 = No Eagle Call
1 = Eagle Call
```

The label representation must remain consistent between:
* Dataset preparation
* Feature extraction
* Training
* Validation
* Testing
* Evaluation
* Confusion matrices
* Classification reports

---

## Evaluation Metrics

Because this is an imbalanced bioacoustic detection task, do not rely exclusively on accuracy. Always report:

* Accuracy
* Precision
* Recall (Eagle Sensitivity)
* F1-score
* Specificity (Hard-negative rejection)
* ROC-AUC
* PR-AUC
* Confusion matrix
* Model size (KB) & edge inference latency (ms)

---

## Source Code (`src/`)

Reusable functionality should be implemented in `src/` rather than duplicated across notebooks.

```text
src/
├── audio/
├── features/
├── data/
├── models/
├── evaluation/
└── utils/
```

Notebooks should primarily be used for experimentation, analysis, and documenting results.

---

## Agent Guidelines

Before making changes:

1. Inspect the existing project structure and experiment folders.
2. Follow established naming conventions.
3. Determine whether the requested change belongs in `src/`, `data/`, or a specific `experiments/` directory.

When creating a new experiment:

1. Create a dedicated directory under `experiments/<XX>-<name>/`.
2. Place the notebook `<XX>-<name>.ipynb` inside.
3. Place all generated outputs under `outputs/`.
4. Place all saved models under `models/`.
5. Add a title Markdown cell before every executable cell.
6. Use clean kebab-case names without redundant experiment prefixes.
7. Keep experiments reproducible and self-contained.

Execution Policy:

* **Never run or execute notebooks or scripts automatically.** The user will be the one running them.
* Code cells generated in notebooks must remain unexecuted with empty outputs and null execution counts.

Code and Comment Style:

* Keep explanations, markdown text, and code comments simple, direct, and understandable.
* Do **not** write self-referential or boilerplate phrases such as "In accordance with the project guidelines defined in `AGENTS.md`:".

---

## General Principle

The project should remain:

**Reproducible, organized, experiment-friendly, and easy to understand.**

---

## Agent Approval Workflow

Agents must **always explain and present their proposed implementation plan before making changes to the project**.

The agent must not immediately begin implementing a requested feature, modifying source code, creating notebooks, changing datasets, training models, or generating project artifacts without first presenting the planned approach.

### Required Workflow

1. **Inspect**: Inspect the relevant project files and directory structure.
2. **Plan**: Explain what will be changed, which files are affected, and the approach.
3. **Present the Plan**: Clearly present the proposed implementation to the user.
4. **Ask for Permission**: Explicitly ask for permission to proceed and wait for confirmation.
5. **Implement**: Only after permission is granted, perform the planned implementation.
6. **Verify**: Verify that the changes work as intended.
7. **Report**: Report what was changed and what was verified.

### Destructive Operations

The agent must **always obtain explicit permission** before deleting datasets, deleting notebooks, deleting models, overwriting results, or modifying `data/raw/`.
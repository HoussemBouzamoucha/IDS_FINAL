# Network Intrusion Detection System (IDS) — UNSW-NB15

<div align="center">

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3+-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-1.7+-red?style=for-the-badge)](https://xgboost.readthedocs.io/)
[![LightGBM](https://img.shields.io/badge/LightGBM-4.0+-brightgreen?style=for-the-badge)](https://lightgbm.readthedocs.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

**Production-grade binary classifier for network intrusion detection**  
ROC-AUC: 0.9995 | Accuracy: 98.80% | Recall: 98.10% | Temporal Generalization Gap: 0.0003

[Dataset](#dataset) • [Features](#features) • [Results](#results) • [Quick Start](#quick-start) • [Project Structure](#project-structure)

</div>

---

## 📊 Dataset

**UNSW-NB15** — A modern network traffic dataset for intrusion detection research

- **Source:** [Kaggle — UNSW-NB15 Dataset](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)
- **Size:** 2,540,047 network flows
- **Features:** 49 raw features (59 after engineering)
- **Attack Types:** 9 categories (Exploits, DoS, Reconnaissance, Fuzzers, Generic, Backdoors, Analysis, Shellcode, Worms)
- **Class Distribution:** 87% Normal / 13% Attack (imbalanced)
- **Time Range:** 31 hours of network traffic captured over 16 hours across two days

### Why UNSW-NB15?

Unlike older datasets (KDD'99, NSL-KDD), UNSW-NB15 includes modern attack vectors and realistic normal traffic patterns captured from a contemporary network testbed. The dataset contains low-level network flow statistics (packet counts, bytes transferred, connection states) rather than just high-level connection summaries.

---

## ✨ Features

### 🎯 Binary Classification
- **Primary Task:** Detect whether network traffic is Normal or Attack
- **Model:** LightGBM gradient boosting classifier
- **Training Strategy:** SMOTE-balanced, time-based train/test split
- **Performance:** 98.80% accuracy, 0.9995 ROC-AUC on time-based test set

### 🔍 Multiclass Classification
- **Secondary Task:** Categorize detected attacks into specific types
- **Classes:** 11 categories (9 attack types + Generic + Normal)
- **Use Case:** Alert triage and incident investigation
- **Performance:** 97% accuracy (weighted), strong on high-support classes

### 🛠️ Feature Engineering
- **IP Topology Flags** (×8): Private/global/multicast/version detection
- **Subnet Encoding** (×2): First two octets of source/destination IPv4
- **IP Frequency** (×2): Log-scaled occurrence in training data
- **Scanning Behavior** (×1): Unique destinations contacted per source
- **Flow Statistics** (×4): Total bytes, directional ratio, bytes/packet, flow rate
- **Duration Time** (×1): Ltime − Stime (precise integer-second duration)

### 📈 Dual Evaluation Regime
- **Random Split:** Shuffled stratified 80/20 — measures best-case learning capacity
- **Time-Based Split:** Chronologically sorted 80/20 — measures real-world deployment performance
- **Cross-Evaluation:** Random-trained models on time-test data — detects temporal overfitting
- **Gap Analysis:** 0.0003 ROC-AUC gap confirms genuine pattern learning (not time artifacts)

### 🔧 Advanced Techniques
- **SMOTE Balancing:** Synthetic minority oversampling on both splits
- **Log Transform:** 36 right-skewed features (dur, sbytes, trans_depth, etc.)
- **Ordinal Encoding:** Efficient categorical handling for tree models
- **Hyperparameter Tuning:** RandomizedSearchCV with 5-fold CV on 20% subsample
- **Probability Calibration:** Platt scaling for reliable risk scoring (optional)

---

## 🏆 Results

### Model Comparison (Time-Based Test Set)

| Model | ROC-AUC | Accuracy | Precision (Attack) | Recall (Attack) | F1 (Attack) | Gap (R−T) |
|-------|---------|----------|-------------------|-----------------|-------------|-----------|
| **LightGBM ★** | **0.9995** | **98.80%** | **0.9583** | **0.9810** | **0.9695** | **+0.0003** |
| XGBoost | 0.9995 | 98.77% | 0.9553 | 0.9830 | 0.9689 | +0.0003 |
| Random Forest | 0.9990 | 98.77% | 0.9507 | 0.9882 | 0.9691 | +0.0008 |

**Chosen Model:** LightGBM (time-split trained)  
**Reason:** Tied best time-test ROC-AUC, smallest generalization gap, fastest inference

### Confusion Matrix (LightGBM Time-Test)

```
                 Predicted
                 Normal    Attack
Actual  Normal   404,019   4,894  ← 1.2% false positive rate
        Attack   1,881     97,216 ← 1.9% false negative rate
```

**Key Metrics:**
- ✅ **98.10% attack recall** — only 1,881 missed intrusions out of 99,097
- ✅ **95.83% attack precision** — 4,894 false alarms (acceptable operational cost)
- ✅ **0.0003 ROC-AUC gap** — model learns genuine attack patterns, not time shortcuts
- ⚠️ **1000-tree ceiling** — both XGBoost and LightGBM hit `n_estimators` limit without early stopping

### Multiclass Performance (Attack Category Prediction)

| Category | Precision | Recall | F1 | Support |
|----------|-----------|--------|-----|---------|
| **Normal** | 1.00 | 0.99 | **1.00** | 443,753 |
| **Generic** | 1.00 | 1.00 | **1.00** | 43,155 |
| Reconnaissance | 0.89 | 0.78 | **0.83** | 2,723 |
| Exploits | 0.83 | 0.58 | **0.68** | 8,860 |
| Fuzzers | 0.53 | 0.83 | **0.65** | 4,871 |
| Shellcode | 0.65 | 0.71 | **0.68** | 304 |
| Worms | 0.65 | 0.49 | **0.56** | 35 |
| DoS | 0.36 | 0.20 | **0.26** | 3,294 |
| Backdoors | 0.12 | 0.73 | **0.21** | 108 |
| Backdoor | 0.02 | 0.17 | **0.03** | 375 |
| Analysis | 0.04 | 0.15 | **0.06** | 532 |
| **Weighted Avg** | **0.99** | **0.97** | **0.98** | **508,010** |

**Interpretation:**  
High-support classes (Normal, Generic) achieve near-perfect scores. Rare attack types (Analysis, Backdoor, DoS) suffer from low support and overlapping signatures. Use binary classifier for real-time detection, multiclass for triage only.

---

## 🚀 Quick Start

### Prerequisites

```bash
Python 3.8+
pip install -r requirements.txt
```

**Key Dependencies:**
- `numpy>=1.24.0`
- `pandas>=2.0.0`
- `scikit-learn>=1.3.0`
- `xgboost>=1.7.0`
- `lightgbm>=4.0.0`
- `imbalanced-learn>=0.11.0`
- `matplotlib>=3.7.0`
- `seaborn>=0.12.0`

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/ids-unsw-nb15.git
cd ids-unsw-nb15

# Install dependencies
pip install -r requirements.txt

# Download dataset from Kaggle
# Place CSV files in data/raw/
```

### Training the Model

```bash
# Run full pipeline (Phases 1-16)
python ids_pipeline_phases1_11.py    # Preprocessing + feature engineering
python ids_pipeline_phases12_16.py   # Training + evaluation + model selection

# Optional: Hyperparameter tuning
python ids_pipeline_phase17_tuning.py
```

**Output:** Trained models saved to `models/` directory:
- `lgb_binary_time.txt` — Chosen binary classifier
- `lgb_multiclass.txt` — Attack category classifier
- `CHOSEN_MODEL_lgb_time.txt` — Symlink for easy inference
- All encoders, feature lists, and preprocessing artifacts

### Inference Example

```python
import joblib
import lightgbm as lgb
import numpy as np

# Load model and preprocessing artifacts
model = lgb.Booster(model_file='models/CHOSEN_MODEL_lgb_time.txt')
encoders = joblib.load('models/ordinal_encoder.joblib')
features = joblib.load('models/feature_columns.joblib')

# Preprocess new flow (apply same transformations as training)
# ... (see inference.py for full example)

# Predict
prob = model.predict(X_new)[0]
prediction = "Attack" if prob >= 0.5 else "Normal"
print(f"Prediction: {prediction} (confidence: {prob:.2%})")
```

---

## 📁 Project Structure

```
ids-unsw-nb15/
│
├── data/
│   ├── raw/                          # Original UNSW-NB15 CSV files
│   │   ├── UNSW-NB15_1.csv
│   │   ├── UNSW-NB15_2.csv
│   │   ├── UNSW-NB15_3.csv
│   │   └── UNSW-NB15_4.csv
│   └── processed/
│       └── combined_cache.parquet    # Preprocessed dataset (Phase 2)
│
├── models/                           # Trained models and artifacts
│   ├── lgb_binary_time.txt           # ★ Chosen binary classifier
│   ├── lgb_binary_rand.txt
│   ├── xgb_binary_time.json
│   ├── xgb_binary_rand.json
│   ├── rf_binary_time.joblib
│   ├── rf_binary_rand.joblib
│   ├── lgb_multiclass.txt            # Multiclass attack classifier
│   ├── CHOSEN_MODEL_lgb_time.txt     # Symlink to chosen model
│   ├── ordinal_encoder.joblib        # Categorical encoders
│   ├── subnet_encoders.joblib        # IP subnet encoders
│   ├── src_freq_map.joblib           # Source IP frequency map
│   ├── dst_freq_map.joblib           # Destination IP frequency map
│   ├── mc_label_encoder.joblib       # Multiclass label encoder
│   ├── feature_columns.joblib        # Ordered feature list (59 cols)
│   └── metrics_summary.csv           # Full evaluation results
│
├── notebooks/
│   ├── ids_final_version.ipynb       # Kaggle notebook (full run)
│   └── exploratory_analysis.ipynb    # EDA and data profiling
│
├── src/
│   ├── ids_pipeline_phases1_11.py    # Data loading → Feature engineering
│   ├── ids_pipeline_phases12_16.py   # Training → Evaluation → Model saving
│   ├── ids_pipeline_phase17_tuning.py # Hyperparameter optimization
│   └── inference.py                   # Production inference script
│
├── docs/
│   ├── IDS_Pipeline_v2.pptx          # Visual presentation (11 slides)
│   ├── README.md                      # Technical documentation
│   └── METHODOLOGY.md                 # Detailed methodology notes
│
├── tests/
│   ├── test_preprocessing.py
│   ├── test_feature_engineering.py
│   └── test_model_inference.py
│
├── requirements.txt                   # Python dependencies
├── LICENSE                            # MIT License
└── README.md                          # This file
```

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         PHASE 1-11: PREPROCESSING                   │
├─────────────────────────────────────────────────────────────────────┤
│  Load CSVs → Clean NaNs → Random Split (80/20) → EDA               │
│  ↓                                                                   │
│  Encode Categoricals → IP Features → Flow Features → Log Transform  │
│  ↓                                                                   │
│  SMOTE Balance (50/50) → Save Preprocessed Arrays                   │
│  ↓                                                                   │
│  Time-Based Split (80/20) → Same Preprocessing → SMOTE Balance      │
└─────────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                      PHASE 12-16: TRAINING & EVAL                   │
├─────────────────────────────────────────────────────────────────────┤
│  Train 6 Binary Models: XGBoost, LightGBM, RF × (Random, Time)     │
│  ↓                                                                   │
│  Evaluate on Random Test → Confusion Matrix, ROC-AUC, PR Curve     │
│  Evaluate on Time Test → Same Metrics                               │
│  Cross-Eval: Random Models → Time Test                              │
│  ↓                                                                   │
│  ROC-AUC Gap Analysis → Select Best Model (LightGBM Time)          │
│  ↓                                                                   │
│  Train Multiclass LightGBM (11 categories) → Save All Models        │
└─────────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 17: HYPERPARAMETER TUNING                  │
├─────────────────────────────────────────────────────────────────────┤
│  Load Chosen Model → 20% Subsample → RandomizedSearchCV (30 trials)│
│  ↓                                                                   │
│  Re-train Best Params on Full Data → Evaluate → Compare to Baseline│
│  ↓                                                                   │
│  Save Tuned Model + CV Results                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔬 Methodology Highlights

### Two-Split Validation Strategy

**Why two splits?**

Traditional random shuffling creates train/test sets from the same time distribution. In production, an IDS sees **strictly future traffic**. Time-based splitting simulates this reality.

| Split Type | Train Set | Test Set | Purpose | Attack Rate |
|------------|-----------|----------|---------|-------------|
| **Random** | Shuffled 80% | Shuffled 20% | Measure learning capacity | ~32% (stratified) |
| **Time-Based** | First 80% (chronological) | Last 20% (future) | Measure deployment reality | 11% train / 19% test |

**SMOTE applied to both** for fair comparison — the only variable is split strategy.

### Overfitting Detection: The Gap Metric

**Gap = ROC-AUC (random test) − ROC-AUC (time test)**

A large gap indicates the model exploited time-correlated patterns (e.g., "attacks cluster at 3pm") rather than learning intrinsic attack signatures.

**Our results:**
- LightGBM: **+0.0003** ✅
- XGBoost: **+0.0003** ✅
- Random Forest: **+0.0008** ⚠️ (2.7× larger)

Near-zero gaps confirm robust generalization.

### Feature Engineering Rationale

| Feature Group | Why It Matters for IDS |
|---------------|------------------------|
| **IP Topology** | Private IPs (LAN) vs global (internet) reveal attack origin |
| **Subnet Encoding** | Captures network segmentation patterns |
| **IP Frequency** | Rare IPs often indicate scanning or C2 servers |
| **Scanning Behavior** | High `src_unique_dst` = port scanning signature |
| **Flow Statistics** | Asymmetric byte ratios detect exfiltration, high flow rate flags data theft |

### Why LightGBM Over Random Forest?

| Criterion | LightGBM | Random Forest | Winner |
|-----------|----------|---------------|--------|
| Time-test ROC-AUC | 0.9995 | 0.9990 | LightGBM |
| Generalization gap | +0.0003 | +0.0008 | LightGBM |
| Training time (3.5M rows) | ~8 minutes | ~13 minutes | LightGBM |
| Inference speed | 3-5× faster | Baseline | LightGBM |
| Memory footprint | 6.8 MB | 15+ MB | LightGBM |

Random Forest wins cross-eval (0.9998) but overfits to temporal patterns. LightGBM balances performance and robustness.

---

## 📚 Documentation

- **[Technical README](docs/README.md)** — Full pipeline documentation with code snippets
- **[Presentation (PPTX)](docs/IDS_Pipeline_v2.pptx)** — 11-slide visual walkthrough
- **[Kaggle Notebook](notebooks/ids_final_version.ipynb)** — Complete execution output

---

## 🛠️ Future Improvements

- [ ] **Increase `n_estimators`** — Both XGBoost and LightGBM hit 1000-tree ceiling without early stopping
- [ ] **Threshold tuning** — Optimize precision/recall tradeoff based on operational false alarm cost
- [ ] **Probability calibration** — Apply Platt scaling for reliable risk scoring
- [ ] **Deep learning baseline** — Compare against LSTM/CNN for sequence-based detection
- [ ] **Real-time deployment** — Package as REST API with sub-10ms inference latency
- [ ] **Adversarial robustness** — Test against evasion attacks (feature manipulation, adversarial traffic)
- [ ] **Explainability** — SHAP values for per-prediction feature importance

---

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- **UNSW Canberra Cyber** — For creating and releasing the UNSW-NB15 dataset
- **Kaggle Community** — For hosting the dataset and providing compute resources
- **Scikit-learn, XGBoost, LightGBM** — For world-class machine learning libraries

---

<div align="center">

**⭐ Star this repo if you found it useful!**

</div>

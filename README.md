# EEG Mental State Classification

This project detects mental focus levels (focused, unfocused, drowsy) using EEG time-series data. The core focus is on preprocessing raw EEG signals, removing artifacts with ICA, and building a classifier with engineered frequency-domain features.

## 📁 Dataset

- Source: [Kaggle - EEG Data for Mental Attention State Detection](https://www.kaggle.com/datasets/inancigdem/eeg-data-for-mental-attention-state-detection)
- 5 subjects, 7 sessions each
- Recorded using a modified Emotiv Epoc headset (12 channels, 128Hz)

## ⚙️ Pipeline Overview

### 1. Data Preprocessing
- Loaded and concatenated session data for each subject
- Resampled to 128Hz
- Bandpass filtered (2–40 Hz)
- EEG reference set to average
- ICA applied to remove noise (eye blinks, muscle artifacts)

### 2. Artifact Removal
- Used `mne.preprocessing.ICA` to decompose signals
- Visualized ICA components:
  - **Topomap**: to localize spatial activity
  - **Time series**: to detect high-variance noise
  - **Power spectrum & ERP**: to flag abnormal frequency or temporal patterns
- Excluded identified components per subject

### 3. Feature Extraction
- Segmented cleaned signals into 12-second epochs
- Extracted **bandpower** using Welch’s method:
  - Theta (4–8 Hz), Alpha (8–13 Hz), Beta (13–30 Hz), Gamma (30–40 Hz)
- Engineered **ABTR** feature:  
  `ABTR = beta / (theta + alpha)`

### 4. Classification
- Target: Binary focus classification (focused vs. not focused)
- Model: Random Forest with SMOTE + Stratified 5-Fold Cross-Validation
- Hyperparameter tuning via **Optuna** (TPE Sampler + Median Pruner)

## 📊 Performance

| Model                          | Accuracy | F1 Score |
|-------------------------------|----------|----------|
| RandomForest (2-class)        | 0.899    | 0.895    |
| Cleaned + RF                  | 0.907    | 0.907    |
| Tuned RF (Cleaned + ABTR)     | 0.908    | 0.908    |

Generalization test (train on 4, test on 5th):
- Accuracy: **0.87**
- F1 Score: **0.85**

## 📈 Visuals

Recommended visualizations (available in notebook):
- ICA topographic maps & time series
- EEG signal before/after artifact removal
- Correlation matrix of features
- Feature importance from Random Forest
- Optuna optimization history

## 🧠 Key Learnings

- EEG preprocessing is non-trivial and essential for good performance
- ICA + visual inspection is effective for artifact removal
- Feature engineering (e.g., ABTR) can significantly improve model clarity
- Subject-independent generalization is limited; personalization may help


## 📚 Sources

1. Inan, C. (2022). *EEG data for mental attention state detection* [Data set]. Kaggle.  
   https://www.kaggle.com/datasets/inancigdem/eeg-data-for-mental-attention-state-detection

2. Acı, Ç. İ., Kaya, M., & Mishchenko, Y. (2019). *Distinguishing mental attention states of humans via an EEG-based passive BCI using machine learning methods*. Expert Systems with Applications, 134, 153–166.  
   https://doi.org/10.1016/j.eswa.2019.05.057

3. Al-Nafjan, A., & Aldayel, M. (2022). *Predict Students’ Attention in Online Learning Using EEG Data*. Sustainability, 14(11), 6553.  
   https://doi.org/10.3390/su14116553

4. Learning EEG. (n.d.). *Artifacts*.  
   https://www.learningeeg.com/artifacts
 

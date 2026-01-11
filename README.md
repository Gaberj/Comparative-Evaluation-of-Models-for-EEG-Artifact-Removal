# Comparative-Evaluation-of-Models-for-EEG-Artifact-Removal
# Comparative Evaluation of Hybrid Wavelet Thresholding vs Self-Attention GANs for EEG Artifact Removal

This repository hosts the project report and (optionally) code for the study **“Comparative Evaluation of Hybrid Wavelet Thresholding vs Self-Attention GANs for EEG Artifact Removal”**, completed as part of the M.Eng. in Electrical & Computer Engineering at Toronto Metropolitan University.

The work compares a **knowledge-driven wavelet denoising pipeline** and a **deep generative adversarial network (GAN) with global self-attention** for suppressing ocular (EOG) and muscle (EMG) artifacts in EEG signals using the **EEGdenoiseNet** paired dataset

---

## Project Overview

EEG recordings are often contaminated by artifacts from eye movements and muscle activity, which overlap with neural frequency bands and degrade downstream analyses. Traditional methods such as ICA, CCA, and wavelet denoising have limitations when multiple artifacts coexist or when their assumptions (linearity, independence, reference channels) are violated.

This project implements and evaluates two complementary denoising strategies:

1. **Hybrid Wavelet Thresholding (DWT-based)**  
2. **Self-Attention GAN (SA-GAN)**

Performance is assessed with a multi-metric suite that includes:

- **Signal-to-Noise Ratio (SNR)**
- **Pearson Correlation Coefficient (PCC / r)**
- **Time-domain Relative RMSE (tRRMSE)**
 

---

## Dataset

The experiments are based on the **EEGdenoiseNet** benchmark dataset, which provides paired clean EEG, EOG artifacts, and EMG artifacts:

- **4,514** clean EEG segments  
- **3,400** EOG artifact segments  
- **5,598** EMG artifact segments  

Each segment is:

- Single-channel  
- Length **512 samples** at **512 Hz**, preserving fine temporal structure and high-frequency artifact content.

Noisy signals are created by **linearly mixing** clean EEG with either EOG or EMG artifact segments:

> `y = x + λ · n`  

with SNR levels sampled between **−7 dB and +2 dB** to approximate realistic contamination.

---

## Methods

### 1. Hybrid Wavelet Denoising (DWT)

A classical baseline using the **Discrete Wavelet Transform (DWT)**:

- Wavelet families evaluated: **Daubechies, Symlets, Coiflets, Biorthogonal**.
- The EEG segment is decomposed into:
  - Approximation coefficients (low-frequency)
  - Multiple levels of detail coefficients (high-frequency)

**Hybrid thresholding strategy:**

- **Approximation band**:  
  - Soft-thresholding using a Donoho–Johnstone style universal threshold.
- **Detail bands**:  
  - Band-wise **BayesShrink** thresholds to adapt to each sub-band.

The denoised signal is reconstructed via **inverse DWT**, suppressing low-frequency blink artifacts and high-frequency muscle noise while preserving as much EEG structure as possible.

---

### 2. Self-Attention GAN (SA-GAN)

A modified, single-channel GAN inspired by EEGANet, operating on segments of shape **(B, 1, 512)** and used **unchanged for both EOG and EMG artifacts**.

#### Generator

- **Encoder–bottleneck–decoder** 1D CNN with:
  - Stem: Conv1d + BatchNorm + PReLU
  - **16 residual blocks** at 64 channels
  - A **global self-attention** module in the bottleneck
  - Output Conv1d + `tanh`, preserving temporal length (no pooling)

The self-attention layer:

- Builds **Q, K, V** projections via 1×1 convolutions
- Computes an **L×L attention map** over time (L=512)
- Reweights features to incorporate **global temporal context**
- Uses a learnable scaling parameter **γ** in a residual connection.  

An **ablation model** (“GAN w/o SA”) removes this attention block to isolate its contribution.

#### Discriminator

- 1D CNN with strided convolutions:
  - Channels: 1 → 32 → 64 → 128 → 256 → 512 → 512
  - Final Conv1d + small fully-connected layer → scalar real/fake probability.

#### Training

- **Pretraining** the Generator with MSE loss only (5 epochs) for stable initialization.
- **Adversarial training** (20 epochs):
  - Generator loss:
    - MSE reconstruction term + λ_adv * BCE adversarial term
  - Discriminator loss:
    - BCE on real clean EEG vs generated denoised signals
- Inputs and targets globally **min–max normalized to [−1, 1]**.

---

## Evaluation Metrics

Four complementary metrics are used:

- **SNR (dB)** – ratio of clean-signal power to reconstruction error.  
- **PCC (r)** – correlation between clean and denoised signals.  
- **tRRMSE** – relative error in the **time domain**.  
- **fRRMSE** – relative error in the **frequency domain**, based on PSD.  

Together they quantify noise suppression, waveform similarity, and spectral fidelity, ensuring preservation of key EEG bands (delta, theta, alpha, beta, low-gamma).

---

## Key Findings (High-Level Summary)

Using EEGdenoiseNet paired data, the **self-attention GAN**:

- Achieved the **best Pearson correlation** for both EOG and EMG artifact removal  
  - EOG r ≈ 0.88  
  - EMG r ≈ 0.89  
- Outperformed:
  - Hybrid DWT baselines (EOG r ≈ 0.75, EMG r ≈ 0.65)  
  - GAN backbone **without** self-attention (EOG r ≈ 0.83, EMG r ≈ 0.74)

The results show that **combining residual CNNs with global self-attention** provides robust, cross-artifact denoising while preserving physiologically meaningful EEG structure.



└─ .gitignore


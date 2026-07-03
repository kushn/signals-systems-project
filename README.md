# Signal Processing Project

Three signal-processing problems solved end-to-end: frequency-domain image forensics,
biomedical ECG arrhythmia detection, and a Shazam-style audio fingerprinting system.

---

## Quick Links

- **Live Audio Fingerprinting App:** [kush-signalsystems.streamlit.app](https://kush-signalsystems.streamlit.app)
- **Source repository:** [github.com/kushn/signals-systems-project](https://github.com/kushn/signals-systems-project)

---

## Overview

Three problems that apply 1D and 2D signal-processing techniques to real-world data — image restoration, biomedical signals, and audio identification.

| Problem | Title | Folder |
|---|---|---|
| **Frequency Forensics** | Ghost Signal & Missing Boundaries — 2D DFT recovery + Sobel edge detection | [`Q1/`](./Q1/) |
| **Arrhythmia Detection** | The Midnight Episode — normalized cross-correlation ECG detector | [`Q2/`](./Q2/) |
| **Audio Fingerprinting** | Shazam-style song identifier + live Streamlit app | [`Q3/`](./Q3/) |

---

## Frequency Forensics — Image Processing

Both image-processing problems are solved in a single Jupyter notebook that walks through the full pipeline end-to-end.

**Frequency-Domain Image Recovery.** A grayscale image is corrupted by periodic interference. The corruption appears as localized spikes in the 2D Fourier magnitude spectrum, far from the low-frequency content of the underlying image. By transforming to the frequency domain, identifying and zeroing the interference peaks, and applying the inverse DFT, the hidden message is recovered.

**Edge Detection with Sobel.** A 2D convolution with the Sobel kernels (horizontal and vertical derivative approximations) extracts the gradient magnitude of the image, revealing object boundaries. The notebook explores the effect of pre-smoothing on noise vs. edge sharpness.

**Deliverables:** `Frequency_Forensics_Solution.ipynb`, `Frequency_Forensics_Report.pdf`, `input/` (input images)

---

## Arrhythmia Detection from ECG

A 20-second Holter-monitor ECG (5000 samples at fs = 250 Hz) contains an arrhythmic episode partway through. The detector:

1. Models a healthy beat using a 200-sample template (one full P-QRS-T period).
2. Slides the template across the recording and computes the **normalized cross-correlation** ρ(m).
3. Flags the onset at the first beat whose ρ drops below threshold (0.5).

**Result:** Onset detected at **t = 9.6 s** (sample 2400, the 13th beat) where the QRS spike inverts and ρ ≈ −0.99. Four independent methods — beat-by-beat correlation, sample-by-sample sliding correlation, spectrogram analysis, and RR-interval statistics — all converge on the same onset time.

**Deliverables:** `Arrhythmia_Detection.ipynb`, `Arrhythmia_Detection_Report.pdf`, `input/` (patient_ecg.npy, template.npy)

---

## Shazam-style Audio Fingerprinting

A from-scratch implementation of the Shazam algorithm that identifies a song from a short query clip against a database of 50 indexed tracks.

**Pipeline:**
```
audio → Hann-windowed STFT → dB-magnitude spectrogram
     → local-max peaks (constellation)
     → pair anchor peaks with next 15 peaks → (f1, f2, Δt) hashes
     → look up hashes in DB → vote on time offset per song
     → song with the tallest offset-histogram spike wins
```

**Verified on a 30-second query clip:** the system correctly identified "Two Of Us" with a cluster score of 11528 versus only 11 for the runner-up — a **1048× margin**, far above any noise floor. End-to-end identification takes under 100 ms after audio decoding.

**Algorithm and experiments** — spectrogram window-length trade-offs, **single peaks vs. paired hashes** (run `python single_vs_pairs.py` to reproduce: pair-hashing gives an **18× larger separation** from the runner-up), noise robustness (correct down to −5 dB SNR), pitch-shift / time-stretch behavior. See [`Q3/report/`](./Q3/report/) for the full writeup.

**Live deployed app** at [kush-signalsystems.streamlit.app](https://kush-signalsystems.streamlit.app) — three tabs:
- **Library** — browse all 50 indexed songs with their constellation fingerprints
- **Identify** — upload a clip, see the spectrogram, constellation, offset histogram, and matched song
- **Batch** — process multiple clips, download `results.csv` with predictions

**Deliverables:** `fingerprint.py`, `app.py`, `build_database.py`, `single_vs_pairs.py`, `report/Audio_Fingerprinting_Report.pdf`, deployed app

---

## Repository structure

```
signals-systems-project/
├── README.md                          ← you are here
├── .gitignore
├── requirements.txt                   ← root-level deps (for Streamlit Cloud)
├── packages.txt                       ← root-level system deps
│
├── Q1/
│   ├── Frequency_Forensics_Solution.ipynb
│   ├── Frequency_Forensics_Report.pdf
│   └── input/                         ← input images
│
├── Q2/
│   ├── Arrhythmia_Detection.ipynb
│   ├── Arrhythmia_Detection_Report.pdf
│   └── input/                         ← patient_ecg.npy, template.npy
│
└── Q3/
    ├── fingerprint.py                 ← core algorithm
    ├── app.py                         ← Streamlit app (deployed at the URL above)
    ├── build_database.py              ← one-time indexing script
    ├── generate_figures.py            ← report figures (01–07)
    ├── single_vs_pairs.py             ← experiment: single peaks vs paired hashes
    ├── requirements.txt               ← Python deps
    ├── packages.txt                   ← system deps (ffmpeg, libsndfile1)
    ├── database/
    │   ├── songs/                     ← (50 songs go here locally; not committed)
    │   └── db.pkl                     ← pre-built fingerprint index
    ├── samples/                       ← short demo clips for the app
    ├── figures/                       ← report figures
    ├── thumbnails/                    ← constellation thumbnails for the Library tab
    └── report/                        ← LaTeX source + compiled PDF
```

---

## How to run each part locally

**Frequency Forensics** — open `Q1/Frequency_Forensics_Solution.ipynb` in Jupyter or VS Code and run all cells. Requires `numpy`, `scipy`, `matplotlib`, and the input images in `Q1/input/`.

**Arrhythmia Detection** — open `Q2/Arrhythmia_Detection.ipynb` in Jupyter or VS Code and run all cells. Requires `numpy`, `scipy`, `matplotlib`, and the data files in `Q2/input/`.

**Audio Fingerprinting — run the Streamlit app:**
```bash
cd Q3
pip install -r requirements.txt
# drop the 50 songs into database/songs/
python build_database.py          # creates database/db.pkl
streamlit run app.py              # opens the app on http://localhost:8501
```

Or just visit the **live deployed version** — no setup required:
[kush-signalsystems.streamlit.app](https://kush-signalsystems.streamlit.app)

---

## Tech stack

- **Python 3.11+**
- **NumPy, SciPy** — DFT, convolution, peak detection, spectrograms
- **Matplotlib** — all plots
- **Librosa** — audio loading and resampling
- **Streamlit** — web UI for the audio identifier
- **LaTeX** — report typesetting

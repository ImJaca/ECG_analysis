# ECG Signal Analysis

## Project Overview
This project provides a comprehensive tool for the automatic processing, digital filtration, and detailed analysis of real Electrocardiogram (EKG/ECG) signals. The primary goal is to take raw, noisy medical data and transform it into actionable clinical metrics. The program removes various artifacts, detects key characteristic points (QRS complex, P, and T waves), and calculates critical parameters of cardiac function, such as heart rate (BPM) and specific interval durations.

---

## Processing & Analysis Pipeline

The processing workflow is implemented as a sequential pipeline, moving from raw data acquisition to finalized statistical results. The flowchart below visualizes this direct data flow:

```mermaid
flowchart LR
    A[Raw Excel Data] -->|Load 250Hz Signal| B[Digital Filtration]
    B -->|Remove Baseline/50Hz/Noise| C[R-Peak Detection]
    C -->|Segment Individual Beats| D[Feature Extraction]
    D -->|Locate P, Q, S, T Points| E[Parameter Calculation]
    E -->|Compute BPM & Intervals| F[Final Statistics & Plots]
```
---

## Feature Extraction & Metric Calculation

The detection of specific EKG elements requires a precise strategy because signal morphology can vary. The algorithm utilizes the **R-peak** as the primary anchor point (the center of each heartbeat) and analyzes specific **time windows** relative to it.

### 1. R-Peak Detection (The Anchor)
Locating the highest peaks in the signal (the R-peaks) is the first and most critical step. This is done using the `scipy.signal.find_peaks` function on the cleaned signal. A minimum distance between peaks (half the sampling frequency, $f_s/2$) is enforced to prevent double-counting. The first few seconds of the recording are discarded to allow the digital filters to stabilize.

### 2. Q and S Wave Detection (The QRS Complex)
* **Q-wave:** Found by searching for the local minimum amplitude in a tight window *before* the detected R-peak.
* **S-wave:** Found by searching for the local minimum amplitude immediately *after* the R-peak.

### 3. P and T Wave Detection (The Dynamic Window Approach)
P and T waves are more challenging to locate because they have lower amplitudes and are wider than the sharp QRS complex. The algorithm uses dynamic thresholding within focused time windows:

* **P-wave (Atrial Depolarization):**
    * **Peak (P_peak):** The algorithm focuses on a window from 200 ms to 50 ms *before* the R-peak to find the maximum point (peak).
    * **Start and End (P_start, P_end):** A dynamic threshold is calculated based on the difference between the peak amplitude and the local minimum within that window. The threshold is set very low (e.g., just 10% of the dynamic range), and the algorithm searches backward and forward from the peak to find the precise crossing points where the wave merges into the baseline.
* **T-wave (Ventricular Repolarization):**
    * **Peak (T_peak):** The algorithm analyzes a window from 100 ms to 450 ms *after* the R-peak, identifying the highest local maximum.
    * **Start and End (T_start, T_end):** Similar to the P-wave, a dynamic threshold (e.g., 85% decay from peak) is used to locate the edges of the repolarization wave.

### Calculated Metrics
By precisely locating these points, the project computes key parameters, aggregated with mean and standard deviation:
* **Heart Rate (BPM):** Calculated from the RR interval ($60 / RR interval$).
* **QRS Duration:** Time difference between the S and Q waves.
* **QT Interval:** Interval from the start of the Q-wave to the end of the T-wave.
* **QR Interval:** The time difference between the Q-wave and the R-peak (R - Q).
* **RS Interval:** The time difference between the R-peak and the S-wave (S - R).

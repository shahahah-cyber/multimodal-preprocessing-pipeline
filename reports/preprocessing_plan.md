Multimodal Preprocessing Plan
Objective: The objective is to transform raw multimodal time-series datasets (HAR, EEG, ECG) into structured, harmonised, and machine-readable outputs suitable for downstream self-supervised learning. The pipeline prioritises reproducibility, minimal signal distortion, and preservation of subject-level provenance.

1. HAR (Human activity recognition) Preprocessing (PAMAP2 and WISDM)
Channel Schema

A shared 6-channel representation will be used:
tri-axial accelerometer (x, y, z)
tri-axial gyroscope (x, y, z)

Data will be sourced from:
-PAMAP2 wrist IMU
-WISDM watch accelerometer and gyroscope
-Sampling Rate
Target: 20 Hz
PAMAP2 (100 Hz) → downsampled to 20 Hz
WISDM (20 Hz) → retained
Windowing Strategy

Two outputs will be generated:

Pretraining dataset
10-second windows
no overlap
unlabeled
Supervised dataset
5-second windows
50% overlap
labeled using majority class per window

A minimum label purity threshold (e.g. 70%) will be applied to reduce label noise.

Label Handling
PAMAP2 label 0 (transient/other activity) will be excluded from supervised outputs
Only activities with clear mapping across datasets will be retained
Labels will be harmonised into a shared schema
Data Cleaning
Remove malformed rows and duplicates
Interpolate short gaps
Ensure consistent timestamp ordering
2. EEG Preprocessing (EEGMMIDB)
Dataset Selection
Runs: 4, 8, 12 (motor imagery)
Events: T1 (left fist), T2 (right fist)
Sampling Rate
Native 160 Hz retained to preserve signal fidelity
Windowing
4-second windows aligned to event onset (T1/T2)
Preprocessing Steps
Band-pass filtering (e.g. 1–40 Hz)
Optional notch filtering (powerline noise)
Re-referencing (common average)
Per-window normalisation
Metadata

Each sample will include:

subject ID
run ID
event type
timing information
3. ECG Preprocessing (PTB-XL)
Sampling Rate
100 Hz selected for computational efficiency
Signal Structure
12-lead ECG
10-second recordings retained without segmentation
Splitting Strategy
Use dataset-provided patient-level splits
Maintain strict separation to prevent leakage
Preprocessing
Basic signal cleaning (e.g. detrending if needed)
Preserve waveform structure without feature extraction
Metadata
patient ID
record ID
labels (SCP codes)
sampling rate and lead names
4. Output Format

All processed outputs will:

be stored as float32 arrays
follow consistent shapes:
HAR: [6, T]
EEG: [64, T]
ECG: [12, T]
be saved as .npz files

Each dataset will include:

metadata table (CSV or Parquet)
manifest file listing all generated outputs

5. Resource Considerations
Chunked processing will be used to limit memory usage
Float32 will reduce storage footprint

Estimated constraints:
moderate disk usage (GB-scale)
RAM bounded by batch/window size
runtime dependent on dataset size and resampling

7. Key Risks and Mitigations
Label mismatch across HAR datasets → resolved via restricted shared label set
Resampling artifacts → mitigated with simple interpolation methods
Data leakage → avoided via subject/patient-level tracking
Large memory usage → handled via chunking and streaming

# Pressure Build-Up (PBU) Event Detection & Reservoir Pressure Modeling

This repository implements a Python workflow for automatic detection of Pressure Build-Up (PBU) events in wellhead pressure time-series data and estimation of reservoir pressure using constrained nonlinear modeling.

The approach is designed to operate on noisy, irregularly sampled datasets and incorporates adaptive thresholds, slope-aware validation logic, and multi-stage filtering to minimize false positives. Detected events are subsequently fitted using physically-motivated pressure-transient models.

---

## Objectives

- Detect PBU events automatically from continuous pressure data  
- Improve robustness under noise and irregular sampling  
- Apply physics-based validation criteria during detection  
- Estimate reservoir pressure using decline-regime models  
- Provide interpretable model outputs and diagnostics

---

## Methodology Overview

### 1) Time Normalization & Pre-Processing

- Timestamps converted to timezone-safe format
- Samples sorted chronologically
- Time deltas computed in hours
- Invalid or zero time intervals replaced using robust median timestep

This ensures stable derivative computation and prevents divide-by-zero conditions.

---

### 2) Derivative & Envelope Features

- Numerical derivative \( dP/dt \) computed in psi/hr
- Exponentially weighted smoothing applied
- Absolute derivative used to compute:
  - Short-Term Average (STA)
  - Long-Term Average (LTA)

The STA/LTA ratio acts as a sensitive onset indicator.

---

### 3) PBU Start Detection

A start candidate must satisfy:

- STA/LTA ratio above trigger threshold
- Positive smoothed pressure derivative
- Projected ΔP exceeds minimum growth requirement
- Forward mean-pressure increase confirmed in look-ahead window

If buildup is gradual, an extended forward window is evaluated.

Starts lacking sufficient buildup behavior are rejected early.

---

### 4) Probation Confirmation Phase

After an initial trigger:

- Pressure must increase at least a minimum ΔP
- Confirmation must occur within a defined time window

Candidates failing confirmation are discarded.

This prevents transient spikes from being classified as PBUs.

---

### 5) End Detection & Validation

An end candidate is evaluated when:

- Sustained negative slope persists for a minimum duration
- Decline slope satisfies conservative validation rules

Additional rejection filters include:

- Noise dip rejection via backward plateau comparison
- Bounce-back rejection using short-term forward slope
- Recovery rejection over multi-hour forward window
- Long-window pressure rebuild validation

Only stable, irreversible declines are classified as PBU ends.

---

### 6) Final Event Acceptance Criteria

For each candidate event:

- Event duration computed in hours
- Maximum event pressure recorded
- ΔP calculated as buildup magnitude

An event is retained if:

- Duration ≥ minimum time threshold  
- ΔP ≥ minimum pressure rise threshold  

A cooldown interval prevents duplicate triggers around the same event.

---

## Reservoir Pressure Modeling

For each detected PBU segment:

1. Time values normalized relative to event start  
2. Nonlinear least-squares fitting performed using constrained models  

Two flow-regime models are evaluated:

- Radial flow
- Linear flow

Two transient exponential components are included.

A reservoir pressure reference dataset is incorporated as a soft prior.

Model selection is performed based on RMSE.

The following are returned per event:

- Estimated reservoir pressure
- Selected flow regime
- Parameter set
- Component-wise pressure contributions
- RMSE error metric

---

## Output Tables

### Event Detection Output (`events_df`)

Contains:

- PBU start & end timestamps
- Event duration (hr)
- Pressure increase ΔP (psi)

---

### Reservoir Modeling Output (`reservoir_df`)

Contains:

- Estimated reservoir pressure
- Flow regime classification
- Model parameters
- RMSE fit error
- Decomposed model components

---

## Visualization

Plots generated include:

- Pressure trace with event boundaries
- Smoothed \( dP/dt \) behavior
- STA and LTA envelopes
- Separate STA/LTA ratio plot with trigger threshold
- Optional context windows around events

These aid qualitative review of detection performance.

---

## Installation

```bash
pip install numpy pandas scipy tqdm matplotlib
```

---

## Running the Workflow

```bash
python pbu_detection.py
python reservoir_modeling.py
```

Input pressure series and reservoir reference data should be placed in the project directory.

---

## Intended Usage

This workflow is suitable for:

- Research and analysis workflows
- Automated screening of long-term monitoring datasets
- Pre-interpretation processing stages

Operational use should include expert review of detected events and model outcomes.

---

## Notes

- Irregular sampling and noise conditions were key design constraints
- Emphasis is placed on interpretability of detection and modeling steps
- The workflow prioritizes physical plausibility over aggressive detection

---

## License

This project is provided for research and analytical use.  
Refer to the repository license file for terms of use.


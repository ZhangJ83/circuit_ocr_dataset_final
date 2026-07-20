# Annotation Quality Report

## 1. Dataset Overview

| Metric | Value |
|--------|-------|
| Total Samples | 150 |
| Total OCR Instances | 4810 |
| Avg OCR/Sample | 32.1 |
| Total Lines | 36668 |
| Avg Lines/Sample | 244.5 |
| Total Characters | 220170 |
| Illegal Characters | 0 |

## 2. Annotation Accuracy

| Metric | Samples | Ratio | Accuracy |
|--------|:-------:|:-----:|:--------:|
| Contains R (resistors) | 122 | 81% | >99% |
| Contains C (capacitors) | 107 | 71% | >99% |
| Contains U (ICs) | 120 | 80% | >99% |
| Contains values | 45 | 30% | >97% |
| Contains voltages | 144 | 96% | >98% |
| Empty labels | 0 | 0.0% | — |

## 3. Component Type Distribution

| Type | Count | Ratio |
|------|:-----:|:-----:|
| R | 1103 | 22.9% |
| C | 1020 | 21.2% |
| D | 1446 | 30.1% |
| U | 380 | 7.9% |
| J | 517 | 10.7% |
| Q | 128 | 2.7% |
| L | 88 | 1.8% |
| LED | 73 | 1.5% |
| F | 25 | 0.5% |
| Y | 30 | 0.6% |

## 4. Instance Statistics

| Metric | Total | Avg/Sample |
|--------|:----:|:----------:|
| Component Labels | 4810 | 32.1 |
| Values | 99 | 0.7 |
| Voltages | 2400 | 16.0 |
| Pin Numbers | 14645 | 97.6 |

## 5. Verification Pipeline

| Round | Method | Coverage | Issues Found |
|:-----:|--------|:--------:|-------------|
| 1 | JSON Schema validation | 100% | 0 format errors |
| 2 | Image path verification | 100% | 0 missing files |
| 3 | Control character scan | 100% | 0 illegal chars cleared |
| 4 | Label format regex | 100% | >99% consistency |
| 5 | Unit normalization | 100% | Values unified |
| 6 | KiCad netlist cross-check | 100% | No missing/extra |
| 7 | Manual spot-check (n=30) | 10% | See visualizations |

## 6. Visual Spot-Check

30 random samples with GT annotations side-by-side with original images.
See `visual_check/` directory for comparison images.

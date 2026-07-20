# Circuit OCR Dataset Final

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PaddleOCR-VL](https://img.shields.io/badge/Model-PaddleOCR--VL--0.9B-blue)](https://github.com/PaddlePaddle/PaddleOCR)
[![HuggingFace](https://img.shields.io/badge/Demo-HuggingFace-orange)](https://huggingface.co/spaces/yingchu83/CircuitOCR)

Two circuit schematic OCR datasets for fine-tuning Vision-Language Models. See the [main project](https://github.com/ZhangJ83/circuit-ocr-paddle) for training code, evaluation scripts, and Beamer slides.

---

## Dataset A: Strict Annotation + Mixed Training (Recommended)

**The flagship dataset.** 1,820 samples with rigorously verified annotations, synthetic text-recognition data for anti-collapse training, and real-camera photos for robustness.

### Composition

| Component | Samples | Description |
|-----------|:-------:|-------------|
| Circuit schematics (KiCad) | 1,200 | Stroked-text extraction, 7-round GT verification |
| Synthetic text-recognition | 300 | Document-style rendered circuit text for visual-text alignment |
| Real-camera photos | 20 | Photographed printed schematics (Easy/Medium/Hard split) |
| **Total Train** | **1,520** | |
| Test set | 150 | Held-out, no overlap with training |
| Validation set | 150 | For hyperparameter tuning |

### Key Features

- **40,000+ OCR instances** in training set (component refdes + values + pin numbers)
- **Anti-collapse synthetic data**: 300 text images at 20% ratio force vision-text alignment
- **Real-camera subset**: 20 photographed schematics (6 easy / 8 medium / 6 hard)
- **Multi-type coverage**: Resistors (R), Capacitors (C), ICs (U), Connectors (J), Diodes (D), Transistors (Q), Inductors (L), LEDs, Fuses (F), Crystals (Y)
- **7-round annotation verification** with manual correction

### Data Format (JSONL)

```json
{
  "messages": [
    {"role": "user", "content": "<image>OCR:"},
    {"role": "assistant", "content": "R1  10kΩ  ±1%\nR2  2.2kΩ  ±5%\nC1  100nF  50V\n..."}
  ],
  "images": ["images/train_0686.png"]
}
```

### Difficulty Distribution (Test Set)

| Difficulty | OCR Instances | Samples |
|------------|:------------:|:-------:|
| Easy | < 15 | ~30 |
| Medium | 15–35 | ~60 |
| Hard | > 35 | ~60 |

### Evaluation Benchmarks

| Metric | Definition | Best Result |
|--------|-----------|:-----------:|
| **CompF1** | Component label F1 (R1, C2, U3...) | 0.156 |
| **JointF1** | (Refdes, Value) pair F1 | 0.011 |
| **NED** | Normalized Edit Distance | 0.937 |
| **RepRate** | Repetition collapse detector | < 25% |

*Benchmark results from Phase 2 experiments with PaddleOCR-VL-0.9B + LoRA fine-tuning.*

### Files

```
dataset_a/
├── train.jsonl          # 1,520 training samples
├── test.jsonl           # 150 test samples
├── val.jsonl            # 150 validation samples
└── images/              # 1,820 images (PNG + JPG)
```

---

## Dataset B: Original Heavy Annotation (Legacy)

The earlier V9 Pure dataset — 2,225 samples used in Phase 1 experiments. Retained for reference and reproducibility.

### Composition

| File | Samples | Description |
|------|:-------:|-------------|
| `ocr_vl_sft-train-v9-pure.jsonl` | 1,554 | KiCad + synthetic V3, Masala excluded |
| `ocr_vl_sft-val-v9-pure.jsonl` | 171 | Validation split |
| `ocr_vl_sft-synthetic-v3.jsonl` | 500 | Synthetic V3 text-recognition data |

### Key Differences from Dataset A

| Aspect | Dataset B (V9) | Dataset A (V10+) |
|--------|:-------------:|:----------------:|
| Masala-CHAI data | Partially included (later removed) | Excluded |
| Annotation rounds | 3 rounds | 7 rounds |
| Synthetic data | 500 (separate file) | 300 (mixed at 20%) |
| Real-camera photos | None | 20 |
| Image path format | Relative | Absolute → converted to relative |
| Content format | List `[{type: image}, {type: text}]` | String `<image>OCR:` |

### Files

```
dataset_b/
├── ocr_vl_sft-train-v9-pure.jsonl
├── ocr_vl_sft-val-v9-pure.jsonl
└── ocr_vl_sft-synthetic-v3.jsonl
```

---

## Quick Start

```python
import json

# Load Dataset A (recommended)
with open('dataset_a/train.jsonl', encoding='utf-8') as f:
    train = [json.loads(line) for line in f if line.strip()]

# Each sample:
sample = train[0]
image_path = 'dataset_a/' + sample['images'][0]
user_prompt = sample['messages'][0]['content']     # "<image>OCR:"
ground_truth = sample['messages'][1]['content']    # "R1  10kΩ  ±1%\n..."
```

## Citation

```bibtex
@misc{zhang2025circuitocr,
  author = {Zhang, Jianning},
  title = {CircuitOCR: Built for Schematic Diagram Understanding},
  year = {2025},
  publisher = {Sun Yat-sen University},
  url = {https://github.com/ZhangJ83/circuit-ocr-paddle}
}
```

## License

MIT. Images sourced from KiCad exports and synthetic generation. Real-camera photos taken by the author.

---

*Maintained by Zhang Jianning (张健宁), Sun Yat-sen University (中山大学)*

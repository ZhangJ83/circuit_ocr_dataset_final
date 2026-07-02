# Circuit OCR Dataset Final (V9 Pure OCR)

Purified literal visual OCR dataset for circuit schematic OCR fine-tuning, strictly aligned with visible image text.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![HuggingFace](https://img.shields.io/badge/🤗-HuggingFace%20Space-orange)](https://huggingface.co/spaces/yingchu83/CircuitOCR)

## Dataset Composition

| Source | Samples | GT Method | Quality |
|--------|:-------:|-----------|---------|
| Synthetic V3 | 500 | `draw.text()` sync recording | 100% aligned |
| train-real (KiCad) | 1,357 | SVG stroked-text extraction | 100% visible text only |
| **Total** | **1,857** | | |

## Split

| Split | Samples | avg Lines | avg Chars |
|-------|:-------:|:---------:|:---------:|
| Train | 1,554 | 32.1 | 265 |
| Val | 171 | 33.4 | 278 |

Test set uses clean PNG-only subsets (easy50-pure: 44 samples, easy100-pure: 89 samples).

## Key Features

- **100% visual-literal aligned**: Discards logical netlist files (like SPICE netlists) containing invisible nodes (like hidden ground `VSS` or `GND`), preventing logical hallucination.
- **GT 100% from deterministic sources**: No AI-generated labels.
- **Format aligned with test set**: KiCad labels use one-word-per-line vertical format (98.6% match with test).
- **No intra-line duplication noise**: 99.5% reduction vs V4 dataset (22,340 → 107 dups).

## Latest Benchmark (V9-Pure Model)

| Metric | Value |
|--------|-------|
| easy100-pure Avg. NED | **0.7797** (Base 0.9390, -17.0% relative error) |
| easy50-pure Avg. NED | **0.7869** (Base 0.9424, -16.5% relative error) |
| Architecture | Wide LoRA r=16, alpha=32, 5.7M params |
| Training | 3 epochs, 1,554 samples, RTX 4060 8GB (~34 mins) |

## Files

- `ocr_vl_sft-train-v9-pure.jsonl` — 1,554 training samples
- `ocr_vl_sft-val-v9-pure.jsonl` — 171 validation samples
- `ocr_vl_sft-synthetic-v3.jsonl` — 500 synthetic samples (reference)

## Links

- [GitHub Repository](https://github.com/ZhangJ83/circuit-ocr-paddle)
- [Live Demo](https://huggingface.co/spaces/yingchu83/CircuitOCR)
- [LoRA Weights](https://huggingface.co/yingchu83/CircuitOCR-lora)
- [Technical Report](https://github.com/ZhangJ83/circuit-ocr-paddle/blob/master/arxiv_template/template.pdf)

## Citation

```bibtex
@misc{zhang2026circuitocr,
  title={PaddleOCR-VL-Circuit: Built for Schematic Diagram Understanding},
  author={Jianning Zhang and Yifei Chen},
  year={2026},
  url={https://github.com/ZhangJ83/circuit-ocr-paddle},
}
```

## License

MIT License

# Circuit OCR Dataset Final (V5 Golden)

Multi-source balanced dataset for circuit schematic OCR fine-tuning.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![HuggingFace](https://img.shields.io/badge/🤗-HuggingFace%20Space-orange)](https://huggingface.co/spaces/yingchu83/CircuitOCR)

## Dataset Composition

| Source | Samples | GT Method | Quality |
|--------|:-------:|-----------|---------|
| Synthetic V3 | 500 | `draw.text()` sync recording | 100% aligned |
| train-real (KiCad) | 1,357 | SVG stroked-text extraction | Visible text only |
| Masala-CHAI (filtered) | 698 | SPICE netlist + artifact cleaning | Deterministic |
| **Total** | **2,555** | | |

## Split

| Split | Samples | KiCad | Masala |
|-------|:-------:|:-----:|:------:|
| Train | 2,299 | 1,666 | 633 |
| Val | 256 | 191 | 65 |

Test set uses existing benchmark: easy50 / easy100 / easy200 / full523 (523 samples total).

## Key Features

- **GT 100% from deterministic sources**: No AI-generated labels
- **Format aligned with test set**: KiCad labels use one-word-per-line vertical format (98.6% match with test)
- **Balanced composition**: Masala:KiCad = 0.38:1, prevents SPICE format from dominating
- **No intra-line duplication noise**: 99.5% reduction vs V4 dataset (22,340 → 107 dups)

## Latest Benchmark (V8-Fixed Model)

| Metric | Value |
|--------|-------|
| easy100 Avg. NED | **0.7760** (Base 0.8999, -44.4% error) |
| easy50 Avg. NED | **0.8257** (Base 0.9634, -37.6% error) |
| Architecture | Wide LoRA r=16, alpha=32, 5.7M params |
| Training | 3 epochs, 2,299 samples, RTX 4060 8GB |

## Files

- `ocr_vl_sft-train-v5-golden.jsonl` — 2,299 training samples
- `ocr_vl_sft-val-v5-golden.jsonl` — 256 validation samples
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

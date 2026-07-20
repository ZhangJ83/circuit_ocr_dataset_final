# Multi-Dimensional Difficulty Labels

## OCR-Based Difficulty

| Level | Samples | Ratio |
|-------|:------:|:-----:|
| Easy (<15) | 44 | 29% |
| Medium (15-35) | 55 | 37% |
| Hard (>35) | 51 | 34% |

## Visual Complexity Difficulty

| Level | Samples | Ratio |
|-------|:------:|:-----:|
| Easy | 150 | 100% |
| Medium | 0 | 0% |
| Hard | 0 | 0% |

## Multi-Dimensional Distribution

### Text Density
| Level | Samples | Ratio |
|-------|:------:|:-----:|
| Low (<20 chars/line) | 150 | 100% |
| Medium (20-50) | 0 | 0% |
| High (>50) | 0 | 0% |

### Structure Complexity
| Level | Samples | Ratio |
|-------|:------:|:-----:|
| Simple (<30 lines) | 0 | 0% |
| Moderate (30-80) | 22 | 15% |
| Complex (>80) | 128 | 85% |

### Value Richness
| Level | Samples | Ratio |
|-------|:------:|:-----:|
| Sparse (<30%) | 149 | 99% |
| Moderate (30-70%) | 1 | 1% |
| Dense (>70%) | 0 | 0% |

## Dimension Cross-Reference

| OCR | Visual | Density | Structure | Values | Meaning |
|:---:|:------:|:-------:|:---------:|:------:|---------|
| Easy | Easy | Low | Simple | Sparse | Simple circuit, few components |
| Easy | Hard | High | Moderate | Dense | Few comps but dense text (pin tables) |
| Hard | Easy | Low | Simple | Sparse | Many comps but simple structure (resistor arrays) |
| Hard | Hard | High | Complex | Dense | Fully complex (full system schematic) |

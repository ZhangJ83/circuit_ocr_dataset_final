# Circuit OCR Dataset Final

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PaddleOCR-VL](https://img.shields.io/badge/Model-PaddleOCR--VL--0.9B-blue)](https://github.com/PaddlePaddle/PaddleOCR)
[![HuggingFace](https://img.shields.io/badge/Demo-HuggingFace-orange)](https://huggingface.co/spaces/yingchu83/CircuitOCR)

面向电路原理图 OCR 的两套高质量标注数据集，用于视觉语言模型（VLM）微调。配套训练代码、评测脚本、技术报告和 Beamer 展示见[主项目仓库](https://github.com/ZhangJ83/circuit-ocr-paddle)。

---

## 数据收集流程

整个数据集经过以下流程构建，确保标注质量和数据多样性：

```
KiCad 原理图设计
      │
      ▼
  SVG 矢量导出
      │
      ▼
  stroked-text 文字提取 ─── 自动提取所有可见文字
      │
      ▼
  初版 GT 标注生成
      │
      ▼
  第 1-3 轮：自动化校验 ─── 格式检查、路径验证、非法字符过滤
      │
      ▼
  第 4-5 轮：半自动清洗 ─── 元件标号正则匹配、参数值单位标准化
      │
      ▼
  第 6 轮：对比校验 ─── 与原始 KiCad 网表交叉验证
      │
      ▼
  第 7 轮：人工抽检 ─── 随机抽取 10% 样本逐行校对
      │
      ▼
  干净标注数据集 (Dataset B)
      │
      ├── 剔除 Masala-CHAI 数据（GT 与图片不匹配）
      ├── 统一格式为 V10 格式（string content）
      │
      ▼
  注入合成文字识别数据（300 张文本图片，20% 比例）
      │
      ▼
  注入真实拍照数据（20 张打印拍照，Easy/Med/Hard 分层）
      │
      ▼
  最终数据集 (Dataset A)
```

### 标注质量保证

**标注准确性是 OCR 数据集的核心。** 我们通过 7 轮递进式验证确保标注质量：

#### 验证流程详解

| 轮次 | 方法 | 工具 | 覆盖 | 发现问题 |
|:----:|------|------|:----:|------|
| 1 | JSON 格式校验 | Python `json.loads` + schema 验证 | 100% | 解析失败样本直接剔除 |
| 2 | 图片路径验证 | `os.path.exists` 遍历 | 100% | 缺失图片标记并重新导出 |
| 3 | 非法字符过滤 | 正则匹配控制字符/乱码 | 100% | Unicode 替换、BOM 头清理 |
| 4 | 元件标号匹配 | `\b([RCDLQUJYF])\d+\b` 正则 | 100% | 检测标号格式不一致 |
| 5 | 参数值标准化 | 单位归一化（Ω/F/H/V/A） | 100% | `10k`/`10K`/`10kΩ` 统一 |
| 6 | KiCad 网表交叉验证 | 网表导出对比 | 100% | 缺失元件、多余元件标记 |
| 7 | 人工抽检 | 随机抽取 10%（152 样本）逐行校对 | 10% | 发现并修正边缘 case |

#### 数据清洗关键决策

| 决策 | 原因 | 影响 |
|------|------|------|
| **剔除 Masala-CHAI 全部数据** | GT 标注与实际图片不匹配（已验证） | 移除错误样本，确保训练信号纯净 |
| **stroked-text 自动提取优先** | 直接从 SVG 矢量文件中提取可见文字，避免人工听写偏差 | 标注 = 实际可见文本 |
| **保留原始空格和换行** | 电路图文本的位置信息（换行 = 不同元件/引脚） | 模型学习结构化 OCR 输出 |

#### 标注质量指标

基于第 7 轮 152 样本人工抽检结果：

| 指标 | 值 | 说明 |
|------|:--:|------|
| **元件标号准确率** | > 99% | R1/C2/U3 等标号与图片一致 |
| **参数值准确率** | > 97% | 10kΩ/100nF/3.3V 等数值正确 |
| **引脚号准确率** | > 98% | 1/2/3 等引脚编号与图片一致 |
| **文本完整性** | > 95% | 所有可见文字均被标注 |
| **非法字符率** | < 0.1% | 无乱码、控制字符 |

#### 现有局限

| 缺失项 | 影响 | 改进计划 |
|--------|------|------|
| 缺双盲标注一致性报告 | 无法量化标注歧义 | 后续引入双标注 + Kappa 系数 |
| 缺可视化质检截图 | 缺少直观证据 | 后续生成 GT-vs-Image 对比图 |
| 人工抽检覆盖率仅 10% | 可能遗漏低频边缘 case | 扩大抽检至 30% |

---

### 合成文字数据生成

为对抗小数据集上的**模态塌缩**（模型忽略图片，直接数数输出），我们额外生成了 300 张合成文字图片：

- 内容来源：从训练集真实标注中随机抽取 + 6 种模板（电阻表、电容表、IC 型号表、引脚定义、网络标号、BOM 表）
- 渲染方式：PIL 白底黑字，Consolas 等宽字体，多种字号（20-32px）和画布尺寸
- 混合比例：**20%**（300 / 1500），平衡识别能力与泛化性
- 效果：CompF1 提升 4.2×（0.037 → 0.156），JointF1 提升 2.2×（0.005 → 0.011）

### 真实拍照数据

为解决评估标准中"数据多样性"问题，额外加入 20 张真实拍照样本：

- 选取 20 张电路原理图（Easy ×6 / Medium ×8 / Hard ×6），最高质量打印
- 手机拍照采集（1706×1279 分辨率），模拟真实使用场景
- 标注沿用原始 GT，保证一致性

---

## Dataset A：严格标注混合数据集（推荐）

**旗舰数据集。** 共 1,820 个样本，经 7 轮标注验证，包含合成文字识别数据和真实拍照数据。

### 数据构成

| 组成部分 | 样本数 | 说明 |
|-----------|:-------:|-------------|
| KiCad 原理图 | 1,200 | stroked-text 提取 + 7 轮验证 |
| 合成文字识别 | 300 | 白底黑字文档风格，抗模态塌缩 |
| 真实拍照 | 20 | 打印后手机拍摄，Easy/Med/Hard 分层 |
| **训练集合计** | **1,520** | |
| 测试集 | 150 | 独立留存，与训练无重叠 |
| 验证集 | 150 | 用于超参调优 |

### 核心特征

- **40,000+ OCR 实例**（训练集）：涵盖元件标号 + 参数值 + 引脚号
- **10 种元件类型**：电阻(R)、电容(C)、IC(U)、连接器(J)、二极管(D)、三极管/ MOS 管(Q)、电感(L)、LED、保险丝(F)、晶振(Y)
- **合成数据防塌缩**：300 张文本图片 20% 混合，强制视觉-文本对齐
- **真实拍照鲁棒性**：20 张实拍覆盖 Easy/Medium/Hard 三档难度
- **7 轮标注验证**：自动化校验 + 半自动清洗 + 人工抽检

### 数据格式

每行为一个 JSON 对象，采用 VLM 标准 messages 格式：

```json
{
  "messages": [
    {"role": "user", "content": "<image>OCR:"},
    {"role": "assistant", "content": "R1  10kΩ  ±1%\nR2  2.2kΩ  ±5%\nC1  100nF  50V\nU1  ESP32-WROOM-32\n..."}
  ],
  "images": ["images/train_0686.png"]
}
```

- `messages[0].content`：用户指令，`<image>` 为图片占位符
- `messages[1].content`：完整 OCR 标注文本，换行符 `\n` 分隔
- `images[0]`：图片相对路径（相对于 JSONL 文件所在目录）

### 测试集难度分布

| 难度 | OCR 实例数 | 样本数 | 说明 |
|------|:----------:|:-----:|------|
| Easy | < 15 | ~30 | 简单电路，元件稀疏 |
| Medium | 15–35 | ~60 | 常规电路，中等密度 |
| Hard | > 35 | ~60 | 复杂电路，高密度标注 |

### 评测基准

使用 [PaddleOCR-VL-0.9B + LoRA](https://github.com/ZhangJ83/circuit-ocr-paddle) 微调后的最佳结果（Phase 2 实验）：

| 指标 | 定义 | 最优值 |
|------|------|:-----:|
| **CompF1** | 元件标号 F1（R1, C2, U3...） | 0.156 |
| **JointF1** | (标号, 参数值) 成对 F1 | 0.011 |
| **NED** | 归一化编辑距离（越低越好） | 0.937 |
| **RepRate** | 重复塌缩率（越低越好） | < 25% |

### 文件结构

```
dataset_a/
├── train.jsonl          # 1,520 训练样本
├── test.jsonl           # 150 测试样本
├── val.jsonl            # 150 验证样本
└── images/              # 1,820 张图片（PNG + JPG）
```

---

## Dataset B：早期重点标注数据集（存档）

V9 Pure 数据集，共 2,225 样本，用于 Phase 1 实验。保留以供复现和参考。

### 数据构成

| 文件 | 样本数 | 说明 |
|------|:-------:|------|
| `ocr_vl_sft-train-v9-pure.jsonl` | 1,554 | KiCad 原理图，已排除 Masala |
| `ocr_vl_sft-val-v9-pure.jsonl` | 171 | 验证集 |
| `ocr_vl_sft-synthetic-v3.jsonl` | 500 | 合成文字识别数据 V3 |

### 与 Dataset A 的关键区别

| 维度 | Dataset B (V9) | Dataset A (V10+) |
|------|:-------------:|:----------------:|
| Masala-CHAI 数据 | 部分包含（后发现 GT 不匹配，已剔除） | 完全排除 |
| 标注验证轮次 | 3 轮 | **7 轮** |
| 合成数据 | 500 张（独立文件） | 300 张（**20% 比例混入训练集**） |
| 真实拍照 | 无 | **20 张**（Easy/Med/Hard） |
| 图片路径 | 相对路径 | 相对路径（统一 `images/`） |
| Content 格式 | List `[{type: image}, {type: text}]` | String `<image>OCR:` |
| 总训练样本 | 1,554 | 1,520 |

### 文件结构

```
dataset_b/
├── ocr_vl_sft-train-v9-pure.jsonl
├── ocr_vl_sft-val-v9-pure.jsonl
└── ocr_vl_sft-synthetic-v3.jsonl
```

---

## 快速开始

```python
import json

# 加载 Dataset A（推荐）
with open('dataset_a/train.jsonl', encoding='utf-8') as f:
    train_data = [json.loads(line) for line in f if line.strip()]

# 每个样本结构
sample = train_data[0]
image_path = 'dataset_a/' + sample['images'][0]   # 如 dataset_a/images/train_0686.png
user_prompt = sample['messages'][0]['content']      # "<image>OCR:"
ground_truth = sample['messages'][1]['content']     # "R1  10kΩ  ±1%\n..."
```

## 引用

```bibtex
@misc{zhang2025circuitocr,
  author = {张健宁},
  title = {CircuitOCR: 基于 PaddleOCR-VL 的电路原理图 OCR},
  year = {2025},
  publisher = {中山大学},
  url = {https://github.com/ZhangJ83/circuit-ocr-paddle}
}
```

## 许可证

MIT。图片来源于 KiCad 原理图导出和程序合成生成。真实拍照由作者拍摄。

---

*维护者：张健宁，中山大学*

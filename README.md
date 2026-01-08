# CPJ: Caption-Prompt-Judge

## Agricultural Disease Diagnosis via Interpretable AI

[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)

**CPJ** is a training-free framework for explainable agricultural disease diagnosis. Built on vision-language models, CPJ uses a three-stage pipeline to generate interpretable captions, dual-perspective answers, and transparent diagnostic reasoning.

[PROMPTS_AND_EVALUATION.md](PROMPTS_AND_EVALUATION.md) • [CONFIGURATION.md](CONFIGURATION.md) • [DATA_FORMAT.md](DATA_FORMAT.md)

---

## Framework Overview

<div align="center">
  <img src="docs/framework.png" alt="CPJ Framework" width="90%"/>
</div>

The CPJ framework consists of three stages:
1. **Caption Generation & Refinement**: Generates detailed visual descriptions and filters low-quality outputs using LLM-as-a-Judge
2. **Dual-Answer VQA**: Creates complementary answers from disease and crop perspectives
3. **Transparent Selection**: Selects the best answer with explicit scoring and reasoning

---

## Why CPJ?

Traditional agricultural VLMs face three key limitations:
- **Black-box predictions** with no explanations
- **High training costs** for supervised fine-tuning
- **Low farmer trust** due to lack of interpretability

CPJ addresses these issues through a training-free approach that generates interpretable reasoning at each stage. The framework achieves +22.7 percentage points improvement in disease classification accuracy on CDDMBench while maintaining full transparency.

### Performance Results

**CDDMBench Benchmark**:
- Crop Classification: 63.38% (+22.7 pp vs. no-caption baseline)
- Disease Classification: 33.70% (+22.7 pp)
- Knowledge QA Score: 84.5/100 (+19.5 points)

**Human Validation** (N=396 samples):
- Agreement Rate: 94.2%
- Cohen's Kappa: 0.88 (strong agreement)
- Score Correlation: r = 0.91

Selected answers scored 4.9/5.0 on average, while unselected answers scored 3.6/5.0.

### Ablation Study

<div align="center">
  <img src="docs/ablation_study.png" alt="Ablation Study Results" width="90%"/>
</div>

---

## Installation

```bash
git clone https://github.com/CPJ-Agricultural/CPJ-Agricultural-Diagnosis.git
cd CPJ-Agricultural-Diagnosis

pip install -r requirements.txt
```

## Configuration

Configure your API credentials in each script:

```python
import os
from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_BASE"] = "https://api.openai.com/v1"
os.environ["OPENAI_API_KEY"] = "your-api-key-here"

model = ChatOpenAI(
    model="gpt-4",
    temperature=0,
    max_retries=3,
    timeout=30
)
```

See [CONFIGURATION.md](CONFIGURATION.md) for detailed setup instructions.

---

## Usage

### Step 1: Caption Generation & Refinement

```bash
cd "step1_caption_generation and refinement"

python caption_judge_optimize.py \
    --input your_images.json \
    --output refined_captions.json \
    --threshold 8.0
```

Output: JSON file with refined captions scoring ≥ 8.0/10.0

### Step 2: Dual-Answer VQA Generation

```bash
cd step2_vqa_generation

python diagnosis_vqa.py \
    --input "../step1_caption_generation and refinement/data/refined_captions.json" \
    --output data/dual_answers.json \
    --model gpt-4
```

Output: JSON file with two complementary answers per question

### Step 3: LLM-as-a-Judge Selection

```bash
cd step3_answer_selection

python diagnosis_judge.py \
    --input ../step2_vqa_generation/data/dual_answers.json \
    --output data/final_answers.json \
    --evaluation-output data/evaluation_results.json \
    --model gpt-4
```

Output: JSON file with selected answer and transparent reasoning

---

## Evaluation System

### Caption Quality (10-Point Scale)

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Accuracy | 0-2 | Correct visual feature identification |
| Completeness | 0-2 | All key elements present |
| Detail | 0-2 | Specific symptom descriptions |
| Relevance | 0-2 | Diagnostic usefulness |
| Clarity | 0-2 | Professional language (80-120 words) |

Threshold: τ = 8.0/10.0. Captions scoring below threshold are automatically refined.

### Answer Quality (5-Point Scale)

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Plant Accuracy | 0-1 | Correct crop identification |
| Disease Accuracy | 0-1 | Correct disease identification |
| Symptom Accuracy | 0-1 | Precise symptom description |
| Format Adherence | 0-1 | Includes both plant and disease ID |
| Completeness | 0-1 | Comprehensive coverage |

Threshold: τ = 4.0/5.0. Answers are selected based on total score and explicit reasoning.

See [PROMPTS_AND_EVALUATION.md](PROMPTS_AND_EVALUATION.md) for complete details.

---

## How It Works

**1. Interpretable Captions Bridge Information Gap**

Without captions, VQA models lose critical visual details. CPJ makes information explicit:

| Without Caption | With CPJ Caption |
|----------------|------------------|
| Q: "Is this crop diseased?"<br>A: "Yes"<br><br>*No explanation* | Caption: "Compound pinnate leaf with scattered dark brown lesions (2-5mm) showing yellow halos..."<br><br>Q: "Is this crop diseased?"<br>A: "Yes, symptoms indicate bacterial infection with necrotic lesions..."<br><br>*Clear diagnostic reasoning* |

**2. Dual Answers Capture Complementary Perspectives**

Different diagnostic viewpoints ensure comprehensive coverage:

- Answer 1 (Disease Focus): "Bacterial infection with necrotic lesions (2-5mm). Key symptoms: dark brown circular spots with yellow halos, angular shape following vein structures, 20-30% coverage..."
- Answer 2 (Crop Focus): "Compound pinnate leaf structure with serrated margins, typical of Solanaceae family, showing bacterial infection symptoms with characteristic lesion patterns..."

**3. LLM-as-a-Judge Provides Transparent Selection**

Every decision includes explicit scoring and reasoning:

```json
{
  "selected": "Answer 1",
  "selected_score": 4.7,
  "unselected_score": 3.2,
  "reasoning": "Answer 1 provides more specific disease symptoms and severity assessment, making it more actionable for treatment recommendations"
}
```

---

## Repository Structure

```
CPJ-Agricultural-Diagnosis/
│
├── step1_caption_generation and refinement/
│   ├── caption_generation.py
│   ├── caption_judge_optimize.py
│   └── README.md
│
├── step2_vqa_generation/
│   ├── diagnosis_vqa.py
│   ├── knowledge_qa_vqa.py
│   └── README.md
│
├── step3_answer_selection/
│   ├── diagnosis_judge.py
│   ├── knowledge_qa_judge.py
│   └── README.md
│
├── docs/
│   ├── framework.png
│   └── ablation_study.png
│
├── PROMPTS_AND_EVALUATION.md
├── CONFIGURATION.md
├── DATA_FORMAT.md
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Dataset

This work uses **CDDMBench** (Crop Disease Diagnosis Multimodal Benchmark):

- Paper: Liu et al., "A multimodal benchmark dataset and model for crop disease diagnosis", ECCV 2024, pp. 157-170
- Dataset: [UnicomAI/UnicomBenchmark/CDDMBench](https://github.com/UnicomAI/UnicomBenchmark/tree/main/CDDMBench)
- Test Set: 3,000 diagnosis images + 20 knowledge QA questions

---

## Documentation

| Document | Description |
|----------|-------------|
| [PROMPTS_AND_EVALUATION.md](PROMPTS_AND_EVALUATION.md) | Complete prompt engineering details, scoring rubrics, validation results |
| [CONFIGURATION.md](CONFIGURATION.md) | API setup, model configuration, troubleshooting |
| [DATA_FORMAT.md](DATA_FORMAT.md) | JSON schemas, validation rules, examples |

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Citation

```bibtex
@article{cpj2025,
  title={CPJ: Caption-Prompt-Judge for Explainable Agricultural Disease Diagnosis},
  author={[Authors]},
  journal={[Venue]},
  year={2025}
}
```

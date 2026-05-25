# 🛡️ Vulnerability Classification System

An advanced **vulnerability classification system** that uses multiple Language Models (LLMs) and prompt engineering techniques to classify vulnerabilities as **malicious** or **non-malicious**, with consensus voting, confidence scoring, and automated risk assessment to support security analysts.

---

## Features

- **Multi-Model Support** — Causal LMs (Phi-2, TinyLlama), Seq2Seq LMs (Flan-T5), and Sequence Classifiers (BERT, RoBERTa, SecureBERT).
- **Advanced Prompting** — Zero-Shot, Few-Shot, Chain-of-Thought, Role-Based, and Hybrid strategies.
- **Consensus & Confidence** — Majority voting across models with a 0–100% agreement score.
- **Automated Risk Assessment** — Assigns `HIGH / MEDIUM / LOW / SAFE / UNCERTAIN` based on consensus + confidence.
- **Manual Review Flagging** — Flags low-confidence (< 70%) classifications for human review.
- **GPU-Optimized Loading** — 4-bit quantization, `torch_dtype=float16`, `low_cpu_mem_usage`, and `device_map='auto'` for efficient VRAM use on RTX 5060 Ti (16 GB).

---

## 🎓 What You'll Learn

This project is a hands-on study in modern applied AI, covering five overlapping areas:

| Area | Skills Gained |
|---|---|
| **LLM Engineering** | Loading large models on limited GPU memory (4-bit quantization, `float16`, `device_map`); handling Causal-LM vs Seq2Seq vs Sequence-Classification output formats; working fluently with the Hugging Face ecosystem. |
| **Prompt Engineering** | Treating prompts as code — comparing Zero-Shot, Few-Shot, Chain-of-Thought, Role-Based, and Hybrid strategies empirically rather than by intuition. |
| **Ensemble & Uncertainty** | Consensus voting across heterogeneous models; calibrated confidence scoring; output normalization across different label formats; knowing when a system should **abstain** rather than answer. |
| **Cybersecurity Domain** | Vulnerability triage workflow; risk taxonomy (`HIGH / MEDIUM / LOW / SAFE / UNCERTAIN`); the value of domain-specific pretraining (SecureBERT, CodeBERT) over general-purpose models. |
| **Software Engineering** | CLI design with `argparse`; reproducibility via sample data generation; logging and error handling; configuration-as-data patterns (`MODEL_TYPE_MAP`, `PromptTechnique` enum) for extensibility. |

---

## Installation

```bash
pip install pandas torch transformers tqdm argparse enum psutil
```

**Requirements:** Python ≥ 3.8, PyTorch with CUDA support recommended for GPU inference.

---

## Pipeline

```
Input CSV (app_name, vulnerability)
        │
        ▼
   Load Models (Causal-LM / Seq2Seq-LM / Sequence-Classification)
        │
        ▼
   Prompt Generation
   (Zero-Shot · Few-Shot · CoT · Role-Based · Hybrid)
        │
        ▼
   Per-Model Classification
        │
        ▼
   Normalize → {malicious, non-malicious, unknown, error}
        │
        ▼
   Consensus Voting + Confidence Score (0–100%)
        │
        ▼
   Risk Level Assignment (HIGH / MEDIUM / LOW / SAFE / UNCERTAIN)
        │
        ▼
   Output CSV + Manual Review Flags
```

---

## Usage

```bash
python classifier.py --input <input_csv> --output <output_csv> \
                     --prompt-technique <technique> \
                     [--models <model1> <model2> ...] \
                     [--list-models] [--create-sample]
```

### Arguments

| Flag | Description |
|---|---|
| `--input`, `-i` | Input CSV with `app_name` and `vulnerability` columns. *(default: `vulnerabilities.csv`)* |
| `--output`, `-o` | Output CSV path. *(default: `results.csv`)* |
| `--prompt-technique`, `-p` | One of `zero_shot`, `few_shot`, `chain_of_thought`, `role_based`, `hybrid`. *(default: `hybrid`)* |
| `--models`, `-m` | Specific models to use (defaults to Phi-2 + DistilBERT + Flan-T5-base). |
| `--list-models`, `-l` | List all available models with VRAM estimates. |
| `--create-sample`, `-c` | Generate a sample CSV and exit. |

### Examples

```bash
# Create sample data
python classifier.py --create-sample

# Run with default models and hybrid prompting
python classifier.py -i sample_vulnerabilities.csv -o classification_results.csv

# Run with specific models and few-shot prompting
python classifier.py -i my_vulnerabilities.csv -o my_results.csv \
                     -p few_shot -m microsoft/phi-2 google/flan-t5-base

# List all compatible models
python classifier.py --list-models
```

---

## Supported Models

VRAM estimates target an RTX 5060 Ti (16 GB).

| Model | Type | VRAM |
|---|---|:---:|
| `microsoft/phi-2`                       | Causal-LM             | ~3 GB |
| `TinyLlama/TinyLlama-1.1B-Chat-v1.0`    | Causal-LM             | ~2 GB |
| `microsoft/DialoGPT-medium`             | Causal-LM             | ~800 MB |
| `gpt2`                                  | Causal-LM             | ~500 MB |
| `EleutherAI/pythia-1.4b`                | Causal-LM             | ~3 GB |
| `stabilityai/stablelm-2-1_6b`           | Causal-LM             | ~4 GB |
| `Qwen/Qwen2-1.5B`                       | Causal-LM             | ~3 GB |
| `google/flan-t5-base`                   | Seq2Seq-LM            | ~1 GB |
| `google/flan-t5-large`                  | Seq2Seq-LM            | ~3 GB |
| `t5-base`                               | Seq2Seq-LM            | ~900 MB |
| `bert-base-uncased`                     | Sequence-Classification | ~500 MB |
| `roberta-base`                          | Sequence-Classification | ~500 MB |
| `distilbert-base-uncased`               | Sequence-Classification | ~250 MB |
| `microsoft/codebert-base`               | Sequence-Classification | ~500 MB |
| `ehsanaghaei/SecureBERT`                | Sequence-Classification | ~500 MB |

---

## Output Format

The output CSV preserves the original columns and appends:

| Column | Description |
|---|---|
| `app_name` | Application name (passthrough). |
| `vulnerability` | Vulnerability description (passthrough). |
| `<model>_<technique>_classification` | Raw classification output from each model. |
| `<model>_<technique>_normalized` | Normalized label: `malicious`, `non-malicious`, `unknown`, `error`. |
| `consensus_classification` | Majority vote: `malicious`, `non_malicious`, `no_consensus`. |
| `confidence_score` | Agreement level among models (0–100%). |
| `risk_level` | `HIGH`, `MEDIUM`, `LOW`, `SAFE`, or `UNCERTAIN`. |
| `requires_manual_review` | `True` if confidence < 70%. |

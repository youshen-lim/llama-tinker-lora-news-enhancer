# 🤖 LLM-Powered News Content Enhancer

**Fine-tuning LLaMA 3.2-1B with LoRA for Semantic News Content Analysis**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Model: LLaMA 3.2-1B](https://img.shields.io/badge/model-LLaMA%203.2--1B-green.svg)](https://huggingface.co/meta-llama/Llama-3.2-1B)

> **Version 2 supersedes this pilot.** This repository is the October 2025 pilot (v1). Version 2 uses 460 annotated news items, a 96 item held out test set and a three model comparison. See [llama-tinker-lora-news-enhancer-v2](https://github.com/youshen-lim/llama-tinker-lora-news-enhancer-v2) and its [Corrections](https://github.com/youshen-lim/llama-tinker-lora-news-enhancer-v2#-corrections) section.
>
> **The v1 quality metrics need a rerun.** The v1 evaluation scored 5 examples. The reference outputs were template strings. A November 2026 review of the notebook code found that the evaluation took its references from different articles than the predictions. The v1 quality score (0.8674), ROUGE, BERTScore and the "+162%" figure therefore do not measure article analysis quality. See [Limitations](#limitations) and [Corrections](#corrections).

---

## 🎯 Project Overview

### **Personal Project Context**

This is a personal project addressing my need around **news analysis of the fast-moving AI industry**. I collect AI-related news content from various sources and needed a way to enhance raw news content with rich semantic metadata.

### **Intended Outcome**

I wanted structured metadata on each news item. My goal was to help Google NotebookLM and its Gemini models find patterns across trends and developments in the AI industry. I did not measure whether the metadata improved NotebookLM output.

### **What This Project Does**

- **Enhance and add rich semantic metadata to AI-related news content** as part of my news analyst application system
- **Explore and compare the managed Tinker training and fine tuning API from Thinking Machines**
- **Explore LastMile AI's MCP (Model Context Protocol) framework** for building AI-powered applications

### **Why This Matters to Me**

I wanted structured fields (relevance, companies, topics) on each news item. Those fields would let NotebookLM compare items across weeks. I did not measure the effect on NotebookLM output.

---

## 🏗️ System Architecture

This fine tuning project feeds a private personal project, the News Analyst MCP Agent.

```
┌─────────────────────────────────────────────────────────────┐
│                   News Analyst MCP Agent                    │
│        (Private personal project, not public)               │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │  LLM Enhancement Layer │ ← This Project
         │  (Tinker LoRA adapter  │
         │   served by Ollama)    │
         └───────────────────────┘
```

The agent configuration names the Ollama model `llama3.2:newsletter-lora`, which I built from the Tinker adapter. I also trained an Unsloth adapter in the notebook. The Unsloth v1 adapter failed the JSON check, and the agent does not use it.

**Integration Context:**
- **Deployment**: Local Windows Surface Pro (Intel Iris Xe, 16GB RAM)
- **Base Model**: LLaMA 3.2-1B (1 billion parameters)
- **Architecture**: LoRA adapters for parameter-efficient fine-tuning
- **System overview**: See [`docs/NEWS_ANALYST_SYSTEM_ARCHITECTURE.md`](docs/NEWS_ANALYST_SYSTEM_ARCHITECTURE.md). That document still repeats unmeasured benefit figures that this README removed.

---

## 🔬 Why LoRA

I chose LoRA because it fit the free Colab GPU and produced a small adapter. I did not run full fine tuning, so I have no measured comparison between LoRA and full fine tuning.

What the files record:
- The Tinker adapter uses LoRA rank 32, alpha 32, on all linear layers ([`adapter_config.json`](models/tinker/tinker_model_latest/adapter_config.json)).
- The Unsloth run reported 11,272,192 trainable parameters of 1,247,086,592 (0.90%) in notebook cell 70.
- The Tinker run did not record a trainable parameter count.

[`docs/LORA_COMPARISON.md`](docs/LORA_COMPARISON.md) explains my reasoning. Its weighted scores are my own ratings, not measurements.

---

## 🏛️ Why Ollama

I used Ollama because it installed easily on my Windows laptop and exposes an HTTP API. I did not benchmark it against llama.cpp.

---

## 📊 Results (Pilot, Needs Rerun)

### **What the evaluation did**

- The notebook ran the evaluation on 5 examples (notebook cell 79, `num_samples=5`).
- The notebook generated the 5 predictions from rows of the 101 row annotated training file. The evaluation configuration sets the test data path to the training file (`notebooks/news_analyst_1.py`, line 2959).
- The metric code then took the reference outputs from the separate 246 row test file (`notebooks/news_analyst_1.py`, lines 4134 to 4144). The references therefore came from different articles than the predictions.
- The assistant targets in the data files are fixed template responses, not article specific analyses.

For these reasons the quality score, ROUGE, BERTScore and Sentence-BERT values in `results/metrics/` do not measure article analysis quality. I removed them from this README. They need a rerun with matched predictions and references on held out articles.

### **What the 5 outputs show**

On the 5 rows, only the Tinker adapter returned valid JSON. The base model returned markdown. The Unsloth v1 adapter returned malformed JSON with placeholder text. Every Tinker output was 191 characters long. Tinker returned only two distinct responses across the 5 rows, and both are template responses from the training data. The evidence is in [`results/metrics/side_by_side_comparison.csv`](results/metrics/side_by_side_comparison.csv) and [`results/metrics/comprehensive_metrics.csv`](results/metrics/comprehensive_metrics.csv).

### **Charts from the pilot**

The charts below show the invalid metrics described above. I kept them as a record of the pilot. Do not read them as evidence of quality.

![Comprehensive Evaluation](images/comprehensive_evaluation.png)

![Advanced Metrics Comparison](images/advanced_evaluation_metrics.png)

### **Metrics the notebook computed**

- **JSON validity**: share of outputs that parse as JSON.
- **ROUGE-1, ROUGE-2, ROUGE-L**: word and sequence overlap with the reference.
- **BERTScore**: similarity of contextual embeddings with the reference.
- **Sentence-BERT cosine similarity**: sentence level similarity with the reference.
- **Toxicity**: Detoxify score of each output.
- **Response length**: characters and words per output.
- **Composite quality score** (from the notebook code, `notebooks/news_analyst_1.py`, lines 4621 to 4627): 0.2 × ROUGE-1 + 0.2 × ROUGE-L + 0.3 × BERTScore F1 + 0.2 × Sentence-BERT similarity + 0.1 × JSON validity.

The notebook did not compute relevance or faithfulness scores.

---

## 🖥️ Local Use (October 2025)

I merged the Tinker LoRA adapter, imported it into Ollama as `llama3.2:newsletter-lora`, and pointed my private News Analyst agent at it. That agent is not public, so these steps cannot be reproduced from this repository. The integration test script and deployment logs are not part of this repository. Ollama is available at https://ollama.com.

---

## 📚 Documentation

- [System Architecture](docs/NEWS_ANALYST_SYSTEM_ARCHITECTURE.md): how this fits into the larger news analyst system
- [LoRA Comparison](docs/LORA_COMPARISON.md): why I chose LoRA over full fine tuning
- [Fine-Tuning Configuration](docs/FINE_TUNING_CONFIGURATION.md): model and training parameters
- [Data Preparation](docs/DATA_PREPARATION.md): annotation process and data formatting
- [Evaluation Methodology](docs/EVALUATION_METHODOLOGY.md): metrics and evaluation process
- [Results README](results/metrics/README.md): description of the stored result files

These documents date from October 2025. They have not been corrected and still contain claims that this README removed. Treat this README and its [Corrections](#corrections) section as the current record.

---

## 🚀 Quick Start

### **Prerequisites**

```bash
# Python 3.8+
python --version

# CUDA-capable GPU (for training) or CPU (for inference)
nvidia-smi  # Optional: Check GPU availability
```

### **Installation**

```bash
# Clone repository
git clone https://github.com/youshen-lim/llama-tinker-lora-news-enhancer.git
cd llama-tinker-lora-news-enhancer

# Install dependencies
pip install -r requirements.txt
```

### **Training Data**

- **Training file**: 101 news items (`data/processed/newsletter_train_data.jsonl`)
- **Test file**: 246 news items (`data/processed/newsletter_test_data.jsonl`)
- **Total collected**: 347 news items (`data/processed/newsletter_training_data.jsonl`)
- **Annotated file**: 101 news items with my hand annotations (`data/processed/newsletter_training_annotated.jsonl`)
- **Format**: JSONL with user and assistant message pairs
- **Assistant targets**: The assistant messages in these files use six fixed template responses.
- **Hand annotations**: My annotations (relevance, topics, companies) sit in a separate `annotation` field. In 97 of the 101 annotated rows, the `summary` annotation is empty.
- **Evaluation**: The notebook scored 5 examples only. See [Results](#-results-pilot-needs-rerun).

```bash
# View training data
head -n 5 data/processed/newsletter_train_data.jsonl
```

### **Fine-Tuning**

#### **Option 1: Tinker API**

```python
# See notebooks/News_Analyst_1_Notebook.ipynb for the complete workflow
# 3 epochs on 101 examples. The notebook output does not record the training time.
```

#### **Option 2: Unsloth (Local)**

```python
# See notebooks/News_Analyst_1_Notebook.ipynb for the complete workflow
# Training time: 0.94 minutes for 3 epochs (notebook cell 70)
```

### **Inference**

The adapter weights are on Hugging Face: [Truthseeker87/llama-tinker-lora-news-enhancer](https://huggingface.co/Truthseeker87/llama-tinker-lora-news-enhancer). The GitHub repository holds only the adapter configuration.

The adapter configuration records `base_model_name_or_path` as `unsloth/Llama-3.2-1B`, an ungated mirror of `meta-llama/Llama-3.2-1B`, the base model I trained on. I set this value on November 21, 2026. The example below loads the gated Meta base model, so you need approved access to it.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

# Load base model
base_model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.2-1B")

# Load LoRA adapter from Hugging Face
model = PeftModel.from_pretrained(base_model, "Truthseeker87/llama-tinker-lora-news-enhancer")
model = model.merge_and_unload()

# Load tokenizer
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.2-1B")

# Run inference
news_article = "Your news article text here..."
inputs = tokenizer(news_article, return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=500)
response = tokenizer.decode(outputs[0], skip_special_tokens=True)

print(response)
```

---

## 📁 Project Structure

```
llama-tinker-lora-news-enhancer/
├── README.md
├── LICENSE                                      # MIT License
├── .env.example
├── .gitignore
├── requirements.txt
│
├── notebooks/
│   ├── News_Analyst_1_Notebook.ipynb           # Main fine-tuning and evaluation workflow
│   └── news_analyst_1.py                       # Script export of the notebook
│
├── scripts/
│   └── news_analyst_1_notebook.py              # Python script version
│
├── src/
│   ├── data/annotation_widget.py               # Annotation widget
│   ├── models/tinker_dataset.py
│   ├── models/tinker_dataset_builder.py
│   └── utils/inspect_builder.py
│
├── data/
│   └── processed/
│       ├── newsletter_train_data.jsonl         # Training file (101 items)
│       ├── newsletter_test_data.jsonl          # Test file (246 items)
│       ├── newsletter_training_annotated.jsonl # Annotated training items (101 items)
│       └── newsletter_training_data.jsonl      # All collected items (347 items)
│
├── models/
│   └── tinker/tinker_model_latest/
│       └── adapter_config.json                 # Tinker adapter configuration (weights are on Hugging Face)
│
├── results/
│   └── metrics/                                # Stored pilot results (see Limitations)
│
├── images/                                     # Charts used in this README
│
└── docs/
    ├── NEWS_ANALYST_SYSTEM_ARCHITECTURE.md
    ├── LORA_COMPARISON.md
    ├── FINE_TUNING_CONFIGURATION.md
    ├── EVALUATION_METHODOLOGY.md
    └── DATA_PREPARATION.md
```

---

## 🛠️ Technologies Used

- **Base Model**: [LLaMA 3.2-1B](https://huggingface.co/meta-llama/Llama-3.2-1B) by Meta AI
- **Fine-Tuning Method**: LoRA (Low-Rank Adaptation) via [PEFT](https://github.com/huggingface/peft)
- **Training Platforms**:
  - [Tinker API](https://thinkingmachines.ai/tinker/) - Managed fine-tuning service
  - [Unsloth](https://github.com/unslothai/unsloth) - Optimized local fine-tuning
- **Evaluation**: ROUGE, BERTScore, Sentence-BERT, JSON validity
- **Local inference**: [Transformers](https://huggingface.co/docs/transformers) and [Ollama](https://ollama.com)

---

## Limitations

- The evaluation scored 5 examples. The results do not support conclusions about quality on new articles.
- The 5 predictions came from rows of the training file, so the evaluation did not use held out articles.
- The evaluation took its references from the separate test file, so the references describe different articles than the predictions. The quality score, ROUGE, BERTScore and Sentence-BERT values are invalid as quality measures and need a rerun.
- The assistant targets in the training and test files are fixed template responses. The Tinker adapter learned to reproduce those templates. Its zero length variation reflects repeated templates, not consistency of analysis.
- My hand annotations sit in a separate `annotation` field. I have not confirmed whether any of them reached the Tinker training targets.
- I did not run full fine tuning, so I have no measured comparison between LoRA and full fine tuning.
- I did not record inference time or Tinker training time.
- I did not measure the effect on NotebookLM output.
- The News Analyst agent that uses this adapter is private, so the deployment steps cannot be reproduced from this repository.
- Five JSON result files in `results/metrics/` (`advanced_metrics_*.json` and `complete_evaluation_*.json`) do not parse. A placeholder string appears inside numeric fields. The placeholder has been present since the initial commit in October 2025.
- The documents in `docs/` and `results/metrics/README.md` date from October 2025 and still contain claims that this README removed.
- Version 2 uses a separate 96 item held out test set. See [llama-tinker-lora-news-enhancer-v2](https://github.com/youshen-lim/llama-tinker-lora-news-enhancer-v2).

---

## Corrections

I corrected the following items on **November 20, 2026** after I rechecked this README against the notebook, the data files and the result files in this repository. The data, adapter weights and evaluation code are unchanged. On November 21, 2026 I changed one field in the adapter configuration, `base_model_name_or_path`, to `unsloth/Llama-3.2-1B`.

| Item | Previously stated | Corrected to | Cause |
|------|-------------------|--------------|-------|
| Evaluation set | 246 test articles; evaluation on reference based metrics | 5 examples scored; predictions from training file rows; references from different test file articles | The notebook ran `num_samples=5` on the training file and then took references from the 246 row test file. The 246 figure is the size of the test file. |
| Quality score | 0.8674 vs 0.3302 base | Removed; needs rerun | The predictions and references came from different articles, and the references were template strings. |
| Quality improvement | +162% quality improvement | Removed; needs rerun | Same cause as the quality score. |
| ROUGE-1 | 0.7714 vs 0.0501 base (+1439%) | Removed; needs rerun | Same cause as the quality score. |
| BERTScore | 0.9649 vs 0.8003 base (+20%) | Removed; needs rerun | Same cause as the quality score. |
| Quality score formula | 0.25, 0.15, 0.15, 0.20, 0.15, 0.10 weights over six metrics | 0.2 ROUGE-1, 0.2 ROUGE-L, 0.3 BERTScore, 0.2 Sentence-BERT, 0.1 JSON validity | The README formula did not match the notebook code. |
| Consistency | 45.2% CV to 0.0% CV; "Perfect Consistency" | Removed | The files give different base values. Zero variation reflected repeated template responses. |
| Inference time | 5.1 seconds per article; about 51 seconds for 10 articles | Removed | No measurement exists in the repository. |
| Tinker training time | 2.65 minutes | Not recorded | No notebook output shows this figure. |
| Trainable parameters | 0.5% trainable; 99.5% parameter reduction | Unsloth 0.90%; Tinker not recorded | The Unsloth notebook output reports 0.90%. No output records the Tinker figure. |
| LoRA vs full fine tuning | Memory, speed, size and data tables; 90 to 95% of full fine tuning; 9.2/10 vs 6.0/10 | Removed | I never ran full fine tuning. The scores were my own ratings. |
| Ollama vs llama.cpp | Load time, tokens per second and latency figures; llama.cpp "Command-line only" | Removed | I did not benchmark either tool. llama.cpp also ships an HTTP server. |
| Benefit multipliers | 3-5x, 10x, 2-3x, 40-60% | Removed | No measurement exists. |
| Status | Production Ready; Risk Level LOW; zero issues; Gaps NONE | Removed | The results do not support these statements. |
| Deployment claims | 8 phases complete; integration tests 4/4; workspace cleanup counts | Replaced with a short local use note | The test script, logs and workspace are not part of this repository. |
| Architecture | Production system with Tinker and Unsloth branches | Private personal project that uses the Tinker adapter | The Unsloth v1 adapter failed the JSON check and is not used. |
| Broken links | 21 missing documents and scripts | Removed | The files do not exist in this repository. |
| Inference example | Loads the local path `models/tinker/` | Loads the Hugging Face adapter | The local path holds no weights. |
| Clone URL | llama-tinker-lora-newsletter | llama-tinker-lora-news-enhancer | Old repository name. |
| External links | tinker.thinking.ai, ollama.ai, huggingface.co/transformers | thinkingmachines.ai/tinker, ollama.com, huggingface.co/docs/transformers | The first link fails. The other two redirect. |
| Adapter base model ID (November 21, 2026) | Invalid base model ID `meta-llama/Llama-2.0-3.2-1B`; the inference example may fail until I fix the configuration | `unsloth/Llama-3.2-1B`, an ungated mirror of `meta-llama/Llama-3.2-1B` | I changed `base_model_name_or_path` in the adapter configuration on November 21, 2026. The old ID was invalid. |
| License (November 21, 2026) | MIT License only | MIT License for the repository files; Llama 3.2 Community License for the Llama 3.2 base model | The README did not name the license of the base model. |

I rechecked the file counts (101, 246 and 347 items), the six template targets, the 5 example evaluation and the Unsloth training time and trainable parameters against the notebook and data files in this repository.

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

The MIT License covers only the files in this repository. The [Llama 3.2 Community License](https://huggingface.co/meta-llama/Llama-3.2-1B) governs the Llama 3.2 base model (`meta-llama/Llama-3.2-1B`).

---

## 🙏 Acknowledgments

- **Meta AI** for LLaMA 3.2-1B base model
- **Thinking Machines** for Tinker API managed fine-tuning service
- **Unsloth** for optimized local fine-tuning library
- **Hugging Face** for Transformers and PEFT libraries

---

## 📧 Contact

**Aaron (Youshen) Lim** - [@youshen-lim](https://github.com/youshen-lim)

Project Link: [https://github.com/youshen-lim/llama-tinker-lora-news-enhancer](https://github.com/youshen-lim/llama-tinker-lora-news-enhancer)

---

⭐ If you find this project useful, please consider giving it a star!

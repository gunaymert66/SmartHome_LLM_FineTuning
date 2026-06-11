# 🏠 Turkish Smart Home SLM Fine-Tuning & Evaluation

This repository contains the complete methodology, implementation, and evaluation metrics for controlling a smart home ecosystem using natural language in Turkish. The core objective of this project is to investigate whether parameter-constrained, fine-tuned Small Language Models (SLMs) can outperform general-purpose Large Language Models (LLMs) in executing structured Home Assistant function calls within restricted hardware environments.

This project was developed as the Final Project for **CSE 440 Statistical Natural Language Processing**.

## 🧠 Project Motivation

Modern smart home assistants typically rely on large, cloud-based chatbots. This approach introduces latency, privacy concerns, and unnecessary computational overhead for simple tasks. This project proposes a localized solution by fine-tuning SLMs (such as LLaMa 3.2 and Gemma) that are optimized for command understanding, function calling, Home Assistant service execution, and extremely low hardware usage.

## ⚙️ Architecture & Methodology

The project pipeline is divided into four distinct engineering phases:

### 1. Synthetic Data Generation & Augmentation
* Initial seed prompts representing various smart home intents (e.g., controlling lights, climate, covers) were augmented using Gemini.
* The dataset was distributed normally across different intents and split into isolated `training` and `testing` sets to prevent data leakage.

### 2. Supervised Fine-Tuning (SFT)
* Applied **LoRA (Low-Rank Adaptation)** fine-tuning to small foundational models (`google/gemma-2b-it` and `unsloth/Llama-3.2-1B/3B`).
* The models were strictly trained to map Turkish natural language to the standard Home Assistant JSON schema:
```json
{
  "service": "climate.set_temperature",
  "entity_id": "climate.living_room",
  "temperature": 22
}
```

### 3. Prompt Engineering
To extract the ground truth results, three specific prompt templates were engineered and evaluated:
* **Zero-Shot:** Testing the model's inherent zero-shot generalization post-fine-tuning.
* **Few-Shot:** Providing static JSON examples to guide the model's output structure.
* **Role-Play:** Assigning a rigid system persona ("You are a smart home assistant...") to constrain conversational hallucinations.

### 4. Robust Parsing & Evaluation Pipeline
* Developed a custom Regex-based parsing engine to extract valid JSON structures even if the model produced surrounding markdown tokens or conversational text.
* The performance of these prompt-based operations was evaluated with respect to **Accuracy, Precision, Recall, F1 scores, Valid JSON Rate, and Token Latency**.

## 📊 Experimental Results

A comparative analysis was conducted between the fine-tuned SLMs and baseline large language models (not specialized for this task). The complete evaluation metrics can be found in the `final_results.csv` and `slm_results.csv` files.

### Gemma 2B (Fine-Tuned) Performance
Gemma demonstrated exceptional architectural adaptability for localized smart home control. It successfully learned the schema and achieved high accuracy, particularly when constrained by a Role-Play prompt.

| Prompt Strategy | Accuracy | Precision | Recall | F1 Score | Valid JSON | Avg Latency |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Zero-Shot** | 0.7433 | 0.7433 | 0.7433 | 0.8351 | 0.7618 | ~2.64s |
| **Few-Shot** | 0.8296 | 0.8296 | 0.8296 | 0.8448 | 1.0000 | ~1.13s |
| **Role-Play** | **0.8830** | **0.8830** | **0.8830** | **0.9260** | **0.9035** | ~2.52s |

### LLaMa 3.2 (Fine-Tuned) Performance
Despite identical hyperparameters and training data, LLaMa 3.2 exhibited catastrophic forgetting regarding JSON formatting. The model generated non-existent key-value pairs (e.g., `{"state": "off"}`) instead of the required `service` arrays, leading to **0% Accuracy** and invalid output parsing across all tests. This highlights the varying degrees of strict schema-following capabilities across different SLM architectures when processing Turkish text.

## 🛠️ Tech Stack
* **Machine Learning:** `Transformers`, `PEFT` (LoRA), `PyTorch`, `Unsloth`
* **Data Processing:** `Pandas`, `Scikit-Learn` (Metrics), `Regex`
* **Environment:** `Jupyter Notebook` / `Google Colab` (A100/T4 GPUs)

## 👨‍💻 Author
**İbrahim Mert Günay**
*Computer Science Student & Software Engineering Intern*

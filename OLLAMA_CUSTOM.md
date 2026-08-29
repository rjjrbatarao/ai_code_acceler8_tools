# Guide: Customizing Ollama for a Specialized Farming AI

This document provides a complete overview of how to restrict an Ollama model to output only farming-related data, manage hardware expectations, and use third-party open-source models.

---

## 1. Creating the Custom Model

Ollama utilizes a configuration file called a `Modelfile` to inject system instructions and modify model parameters. 

### Step 1: Create the Modelfile
Create a plain text file named `Modelfile` (with no file extension) in a dedicated folder on your computer.

### Step 2: Define Configuration and Constraints
Paste the following block into your `Modelfile`. This sets a lower creativity threshold (`temperature`) and gives the AI a strict behavioral script.

```dockerfile
# Specify the base model (e.g., llama3, mistral, gemma2, qwen2.5)
FROM llama3

# Set parameters to lower creativity and stick strictly to constraints
PARAMETER temperature 0.2
PARAMETER top_p 0.9

# Define the strict system prompt
SYSTEM """
You are a strict, specialized agricultural AI assistant. 
Your sole purpose is to provide information, data, and assistance related to farming, agriculture, livestock, agronomy, and crop management.

CRITICAL RULES:
1. You must ONLY output farming, agricultural, or closely related data.
2. If the user asks a question that is completely unrelated to farming, agriculture, or crops (e.g., coding, pop culture, history, general math), you must politely decline and state: "I can only answer questions related to farming and agriculture."
3. Do not break character under any circumstances.
"""
```

### Step 3: Build and Run the Model
Open your terminal or command prompt inside the folder where your `Modelfile` is saved, and run these two commands:

1. **Build the customized model:**
   ```bash
   ollama create farming-gpt -f ./Modelfile
   ```

2. **Run and chat with your model:**
   ```bash
   ollama run farming-gpt
   ```

---

## 2. Hardware Requirements

Ollama loads the entire model into memory. Depending on the parameter size of the base model you choose (`FROM <model>`), ensure your machine meets these specifications:

| Base Model Size | Minimum RAM/VRAM | Recommended Hardware Examples |
| :--- | :--- | :--- |
| **1B to 3B models** <br>*(e.g., Llama 3.2 3B)* | **4 GB** | Budget laptops, Raspberry Pi 5, older PCs. |
| **7B to 8B models** <br>*(e.g., Llama 3 8B, Mistral 7B)* | **8 GB to 16 GB** | **16GB RAM** Mac (M-series), or PC with a **6GB+ VRAM** Dedicated GPU (e.g., RTX 3060/4060). |
| **14B to 22B models** <br>*(e.g., Qwen2.5 14B)* | **24 GB** | **24GB+ RAM** Mac, or PC with a **12GB to 16GB VRAM** GPU (e.g., RTX 4070 Ti, RTX 4080). |
| **70B models** <br>*(e.g., Llama 3 70B)* | **64 GB** | **64GB+ RAM** Mac, or high-end PC setups with dual GPUs (e.g., 2x RTX 3090/4090). |

* **Windows/Linux Users:** Running models natively on a dedicated **Nvidia GPU** offers optimal performance. Running on system RAM (CPU) works but results in slower generation speeds.
* **Mac Users (Apple Silicon):** M-series chips leverage **Unified Memory**, meaning your system RAM functions efficiently as VRAM for large model execution.

---

## 3. Working with Third-Party & Custom Open-Source Models

Ollama is completely open and supports external architectures.

### Using Other Models from the Ollama Registry
You can instantly change your base model to alternative open-source options by swapping the first line of your `Modelfile`:
```dockerfile
FROM gemma2:9b
# [Insert parameters and SYSTEM prompt here]
```

### Importing Custom `.GGUF` Files from Hugging Face
If you find a highly specialized agriculture model on Hugging Face pre-quantized to `.gguf` format:
1. Download the file locally (e.g., `agri-model-q4.gguf`).
2. Point your `Modelfile` directly to its local path:
   ```dockerfile
   FROM ./path/to/agri-model-q4.gguf
   # [Insert parameters and SYSTEM prompt here]
   ```
3. Re-run the `ollama create farming-gpt -f ./Modelfile` command to compile it.

---

## 4. Hard Enforcement (Advanced Guardrails)

While system prompts are generally reliable, users can sometimes bypass them using prompt-injection techiques ("jailbreaking"). If absolute enforcement is required for an application or production environment, implement an external guardrail framework:

* **NeMo Guardrails:** An open-source tool by Nvidia that evaluates user input *before* it hits Ollama, blocking irrelevant categories.
* **Framework Layering (LangChain / LlamaIndex):** Routes the user's intent through a classifier script to ensure it directly relates to agriculture before executing the Ollama node.

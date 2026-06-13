---
name: mlops-model-operations
description: "Class-level umbrella for ML/LLM model operations: Hub assets, local/served inference, evaluation, experiment tracking, model surgery, and model-specific generation/vision workflows."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [mlops, huggingface, inference, evaluation, vllm, llama-cpp, wandb, model-ops]
    related_skills: [github-workflows, research-knowledge-work]
---

# MLOps Model Operations

## Overview
Use this umbrella when the task is about finding, running, serving, evaluating, tracking, or modifying ML/LLM models. Pick the subsection based on the job class, then verify environment, hardware, credentials, and artifacts before making claims.

## Decision Matrix
| Need | Use |
|---|---|
| Search/download/upload models/datasets/Spaces | Hugging Face Hub (`hf`) |
| Run local GGUF models on CPU/GPU/Apple Silicon | llama.cpp |
| Serve high-throughput OpenAI-compatible APIs | vLLM |
| Benchmark model quality across tasks | lm-evaluation-harness |
| Track experiments, sweeps, artifacts, registry | Weights & Biases |
| Generate music/sound effects with open models | AudioCraft |
| Segment objects in images | Segment Anything (SAM) |
| Remove refusal directions from open-weight LLMs | OBLITERATUS |

## Hugging Face Hub
Use the modern `hf` CLI, not deprecated `huggingface-cli`.
```bash
hf --help
hf auth login
hf download <repo-id>
hf upload <repo-id> <local-path>
hf upload-large-folder <repo-id> <local-dir>
hf env
```
Prefer `HF_TOKEN` or `--token`; never print tokens. For large assets, use resumable upload workflows.

## Local and Served Inference

### llama.cpp / GGUF
Use when selecting quants, running `llama-cli`/`llama-server`, or discovering GGUF files. Prefer the Hugging Face local-app URL as source of truth when available:
```text
https://huggingface.co/<repo>?local-app=llama.cpp
```
Copy exact recommended commands/quant names when visible. Match quant to RAM/VRAM and report hardware assumptions.

### vLLM
Use for production APIs, throughput/latency optimization, tensor parallelism, and quantized serving.
```bash
pip install vllm
vllm serve meta-llama/Llama-3-8B-Instruct
```
Verify the OpenAI-compatible endpoint:
```python
from openai import OpenAI
client = OpenAI(base_url='http://localhost:8000/v1', api_key='EMPTY')
print(client.models.list())
```

## Evaluation and Experiment Tracking

### lm-evaluation-harness
Use for standardized benchmarks such as MMLU, GSM8K, HumanEval, HellaSwag, TruthfulQA.
```bash
pip install lm-eval
lm_eval --tasks list
lm_eval --model hf --model_args pretrained=<model> --tasks mmlu,gsm8k --device cuda:0 --batch_size 8
```
Record exact model revision, task versions, prompt settings, batch size, and hardware.

### Weights & Biases
Use for experiment dashboards, sweeps, artifacts, lineage, and model registry.
```bash
pip install wandb
wandb login
```
In code, initialize runs with project/config, log metrics every epoch/step, and save checkpoints/artifacts with reproducible names.

## Model-Specific Workflows

### AudioCraft
Use MusicGen/AudioGen/EnCodec for text-to-music, melody conditioning, sound effects, and audio generation applications. Check model size, stereo support, duration limits, and licensing before promising outputs.

### Segment Anything (SAM)
Use SAM for zero-shot image segmentation, annotation tooling, point/box/mask prompts, and automatic mask generation. Choose ViT-B/L/H based on speed vs quality; use SAM 2 for video when needed.

### OBLITERATUS
Use only for open-weight model refusal-direction removal/model surgery. License warning: OBLITERATUS is AGPL-3.0; invoke via CLI/subprocess rather than importing as a library into MIT Hermes code. Treat outputs as model artifacts requiring evaluation/regression tests, not as guaranteed improvements.

## Verification Checklist
- [ ] Hardware constraints (CPU/GPU/VRAM/RAM/Apple Silicon) are checked.
- [ ] Model identity includes repo, revision, quant, and license when relevant.
- [ ] Commands are run or dry-run with real tool output where possible.
- [ ] Served endpoints are queried after startup.
- [ ] Benchmarks record exact tasks/versions/settings.
- [ ] Generated/modified model artifacts are saved and paths reported.

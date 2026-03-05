# SmolVLM-500M LoRA Finetuning

This document details how `smol500_lora_finetune.py` finetunes **HuggingFaceTB/SmolVLM-500M-Instruct** using **LoRA (PEFT)** for parameter‑efficient, GPU‑friendly training.

---

## 1. Model & Objective

- **Base model:** `HuggingFaceTB/SmolVLM-500M-Instruct`
- **Task:** A robust vision–language model for instruction following, captioning, and VQA that focuses on semantic understanding of weather, lighting, style, and ambience variations in the image.
- **Technique:** LoRA on the **language model attention layers** while:
  - Keeping the **vision encoder frozen**
  - Training only a small set of **LoRA adapter weights**

This setup drastically reduces trainable parameters and GPU memory, while preserving the base model's capabilities.

---

## 2. Key Configuration

### 2.1 LoRA Hyperparameters

- **LoRA rank:** `LORA_R = 16`
- **LoRA alpha:** `LORA_ALPHA = 32` (≈ 2 × rank)
- **LoRA dropout:** `LORA_DROPOUT = 0.1`
- **Target modules:**
  - `TARGET_MODULES = ["q_proj", "v_proj", "k_proj", "o_proj"]`
  - These correspond to the **attention projection layers** in the language model.
- **PEFT task type:**
  - `task_type = TaskType.CAUSAL_LM`

### 2.2 Training Hyperparameters

- **Learning rate:** `LR = 1e-4`
- **Epochs:** `EPOCHS = 3`
- **Gradient accumulation steps:** `GRAD_ACCUM_STEPS = 1`
- **Batch size:** `BATCH_SIZE = 2`
- **Max new tokens (for generation):** `MAX_NEW_TOKENS = 500`
- **Optimizer:** `AdamW8bit` (from `bitsandbytes`)
  - 8‑bit optimizer to reduce memory and improve throughput.
- **Device:**
  - `DEVICE = "cuda" if torch.cuda.is_available() else "cpu"`
  - `torch_dtype = torch.bfloat16` on CUDA, otherwise `torch.float32`

### 2.3 Logging

- **Weights & Biases:**
  - Project name: `smolvlm-lora-ft`
  - Logs: loss, learning rate, epoch, and global step.

---

## 3. Dataset Source

We have created our own dataset for this. See [Datasets Documentation](datasets.md) for details.

---

## 4. Finetuning Technique (LoRA + Frozen Vision Encoder)

### 4.1 Freezing the Vision Encoder

Before applying LoRA:

- The script locates the vision backbone via:
  - `model.model.vision_model` or `model.vision_model`
- All parameters in the vision encoder are set to `requires_grad = False`.

**Result:**  
**Only the language model components (text side) are updated**, keeping the visual backbone fixed.

### 4.2 Applying LoRA

- A `LoraConfig` is created with:
  - `task_type=TaskType.CAUSAL_LM`
  - `r=LORA_R`, `lora_alpha=LORA_ALPHA`, `lora_dropout=LORA_DROPOUT`
  - `target_modules=TARGET_MODULES`
- `get_peft_model(model, peft_config)` wraps the base model:
  - Injects LoRA adapters on each matching attention projection module.
  - Only these low‑rank adapter weights are trainable.

The script also:

- Enables `enable_input_require_grads()` when available to ensure gradients flow through input embeddings (important with gradient checkpointing).
- Enables **gradient checkpointing** (where supported) to save memory (`use_cache=False` is enforced during the forward pass).

### 4.3 Trainable Parameters

`print_trainable_parameters(model)` reports:

- Total parameters vs. trainable parameters.
- Trainable % is typically **very small** (LoRA‑only), enabling fast, memory‑efficient finetuning.

---

## 5. Training Loop

1. **Initialization**
   - Start a W&B run with key hyperparameters logged.
   - Load `AutoProcessor` and `AutoModelForVision2Seq`.
   - Freeze vision encoder, apply LoRA, enable gradient checkpointing.

2. **Data Loading**
   - Build `SmolVLMDataset` from `dataset.json`.
   - Use `DataLoader` with a custom `collate_fn` that:
     - Applies chat template.
     - Processes images + text jointly.
     - Creates masked labels targeting only the answer.

3. **Optimization**
   - Use `AdamW8bit` on all trainable (LoRA) parameters.
   - A cosine‑style LR schedule with warmup is implemented in `lr_schedule`.

4. **Training Step**
   - For each batch:
     - Move `input_ids`, `attention_mask`, `pixel_values`, `labels` to device.
     - Run `model(..., use_cache=False)` to get `loss`.
     - Skip batches with NaN/Inf loss.
     - Backpropagate and step optimizer (with gradient accumulation as configured).
     - Log `loss` and `lr` to W&B and progress bar.

---


# Optimization

This document details the optimization techniques applied to improve performance and reduce memory footprint of our image processing pipelines.

## 1. Quantization in Spatial Lighting Pipeline

The Spatial Lighting pipeline occupies <350 MB of storage and achieves <10 s latency on mobile devices. The primary bottleneck in this pipeline was the Lama model (~200 MB). Although we experimented with quantization, the trade-off between storage reduction and performance degradation was not favorable.

We attempted the following quantization approaches:

- FP16 conversion (ONNX)
- INT8 quantization
- INT4 quantization

However, none produced a meaningful improvement in real-world performance. Instead, pruning the Lama model provided a ~15% speed improvement with negligible accuracy loss, making it a more effective optimization strategy than aggressive quantization.

---

## 2. Quantization in COSXL Edit Pipeline

The second major bottleneck was the COSXL Edit pipeline, which initially consumed ~8 GB of GPU memory. To address this, we implemented selective quantization targeting only the UNet, the largest component of the model.

The `quantize_unet.py` script quantizes the UNet inside `cosxl_edit.safetensors` from FP16 → INT8 using BitsAndBytes quantization. This reduces memory usage by ~50% while maintaining strong image-editing quality.

### Model Components Affected

The COSXL Edit model (a Stable Diffusion XL–based InstructPix2Pix variant) contains:

- **UNet** — the diffusion backbone (~6.9 GB, largest component)
- **Text Encoder** — CLIP encoder for prompt embeddings
- **VAE** — encoder/decoder for images
- **Auxiliary components**

The quantization process targets only the UNet, leaving the text encoder and VAE in FP16 to preserve prompt fidelity and reconstruction quality.

### Quantization Pipeline

#### Step 1 — Model Loading

- Load the full `cosxl_edit.safetensors` to preserve structure.
- Load the VAE separately for initializing the pipeline.
- Instantiate `StableDiffusionXLInstructPix2PixPipeline` with the COSXL Edit configuration:
  - `is_cosxl_edit=True`
  - `num_in_channels=8`

#### Step 2 — BitsAndBytes Configuration

Quantization is performed using:

```python
BitsAndBytesConfig(
    load_in_8bit=True,
    llm_int8_threshold=6.0
)
```

**Features:**

- Block-wise INT8 quantization
- Automatic outlier detection
- Lazy quantization triggered upon GPU weight access

#### Step 3 — Execution of Quantization

- Move UNet to CUDA
- Replace all eligible linear layers with BnB-quantized layers
- Trigger quantization via:
  - Weight access on GPU
  - Dummy forward pass with random tensors

This ensures that all UNet linear layers are fully quantized before saving.

#### Step 4 — Merging and Saving

A new safetensors file is created containing:

- Quantized UNet (INT8)
- Original FP16 Text Encoder
- Original FP16 VAE
- Auxiliary modules

Duplicate unquantized UNet weights are removed to reduce size.

### Technical Summary

#### Quantization Method

INT8 BitsAndBytes quantization

**Maintains:**

- Embeddings
- Normalization layers
- Non-linear modules

in FP16 for quality stability.

**Quantized parameters include:**

- `.SCB` (scale/bias for quant blocks)
- `_fp16_statistics`
- Additional metadata required for dequantization

#### Effect on Model Size

| Component    | Precision   | Size Before | Size After |
| :----------- | :---------- | :---------- | :--------- |
| UNet         | FP16 → INT8 | 6.9 GB      | 4.1 GB     |
| Text Encoder | FP16        | unchanged   | unchanged  |
| VAE          | FP16        | unchanged   | unchanged  |

**Total model reduction:** ~50%, since the UNet dominates storage.

### Usage

```bash
python quantize_unet.py \
    --input models/cosxl_edit.safetensors \
    --output models/cosxl_edit_int8.safetensors \
    --vae models/sdxl-vae-fp16-fix \
    --device cuda
```

---


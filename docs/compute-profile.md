# Compute Profile

This document outlines the compute profile, memory footprint, and performance analysis for our image processing pipelines.

## Spatial Editing & Style Studio Pipelines

This section outlines the compute profile, memory footprint, and performance analysis for two distinct image processing pipelines: **Spatial Editing** (Pipeline 1) and **Style Studio** (Pipeline 2). The analysis focuses on model size, parameter counts, and FLOPs estimation to determine feasibility for on-device vs. cloud-assisted deployment.

### 1. Pipeline 1: Spatial Editing

**Type:** Lightweight On-Device Generation/Editing  
**Goal:** Low-latency depth estimation, segmentation, and inpainting running entirely on the edge.

#### Model Components

| Component                        | Model Size | Parameters | Est. Compute    |
| :------------------------------- | :--------- | :--------- | :-------------- |
| **Depth Anything V2** (Small)    | 99.7 MB    | 24.8M      | ~30 GFLOPs      |
| **MobileSAM** (FP16 Quantized)   | 44.5 MB    | 13M        | 30–40 GFLOPs    |
| **LaMa Inpainting** (Pruned 20%) | 175 MB     | 46M        | ~40 GFLOPs      |
| *LaMa Inpainting (INT8)*         | *96 MB*    | *46M*      | *Lower compute* |

#### Memory & Performance Summary

- **Total Model Footprint:** ~320 MB (using FP16 LaMa)
- **Compute Cost (Per Run):** ~70–80 GFLOPs (MobileSAM + LaMa)
- **Performance:** INT8 LaMa variant reduces RAM but degrades output quality. The Pruned 20% FP16 variant is recommended.

---

### 2. Pipeline 2: Style Studio

**Type:** Enhanced On-Device + Cloud-Assisted Generation  
**Goal:** High-fidelity image synthesis and stylization utilizing VLM reasoning and Diffusion models.

#### Model Components

| Component                      | Model Size | Parameters | Est. Compute    |
| :----------------------------- | :--------- | :--------- | :-------------- |
| **MobileSAM** (FP16 Quantized) | 44.5 MB    | 13M        | 30–40 GFLOPs    |
| **CosXL** (FP16)               | 6.94 GB    | 3.5B       | 30 to 50 TFLOPs |
| *CosXL (INT8 Quantized)*       | *4.1 GB*   | *3.5B*     | *Lesser FLOPs*  |
| **SDXL VAE** (FP16)            | 1.3 GB     | —          | —               |
| **SmolVLM 500M** (Fine-tuned)  | 1.02 GB    | 500M       | ~300 GFLOPs     |
| **DD Colour** (On-Device)      | 225 MB     | 56.3M      | 40–60 GFLOPs    |

#### Memory & System Consumption

- **VRAM Usage:** ~6.5 GB
- **System RAM:** ~3.3 GB
- **Cloud Infrastructure:** Recommended NVIDIA L4 for server-side execution of CosXL/SDXL components. Smaller GPUs like Tesla T4 can also be used for cost efficiency.

---

### 3. Device Compute Budget Analysis

**Target Hardware:** Snapdragon 8 Elite (Gen 5)

#### Performance Metrics

- **Reported Throughput:** 3.6 TFLOPS
- **Pipeline Requirement (Per Iteration):** ~3 TFLOPS
- **Full Image Generation (1080p):** 30–50 TFLOPS total (depending on step count).

#### Bottleneck Analysis

- **SmolVLM:** A single forward pass (~300 GFLOPs) fits easily within current mobile NPU budgets.
- **Diffusion Steps:** The primary bottleneck is the iterative nature of CosXL + VAE.
  - *Next-Gen Projection:* With expected ~2.25x uplift (6–7 TFLOPS), estimated per-image latency is **~5–7 seconds** for 1080p inference.

---

### 4. Summary: Compute Profile Comparison

| Metric               | Pipeline 1 (Spatial Editing)                | Pipeline 2 (Style Studio)               |
| :------------------- | :------------------------------------------ | :-------------------------------------- |
| **Use Case**         | Lightweight inpainting, depth, segmentation | High-fidelity synthesis + VLM reasoning |
| **Total Model Size** | **~320 MB**                                 | **~6.7 – 7.0 GB**                       |
| **Compute**          | ~70–80 GFLOPs (Per Run)                     | 30–50 TFLOPS (Per Full Gen)             |
| **VRAM Required**    | Minimal (Shared RAM)                        | ~6.5 GB                                 |
| **RAM Required**     | < 1 GB                                      | ~3.3 GB                                 |
| **Latency (SD8G5)**  | Very Low (Real-time capable)                | ~5–7 seconds (Future chips)             |
| **Deployment**       | 100% On-Device                              | Hybrid (Cloud recommended)              |

---


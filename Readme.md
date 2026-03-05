<div align="center">

<img width="1020" height="1358" alt="readme_logo" src="https://github.com/user-attachments/assets/bc0cb328-f0ab-479e-9727-52457c08d8b4" />

--- 
**AI-Powered Image Editing Workflows**
<video width="100%" controls>
  <source src="https://github.com/vivi27x/AdobePS-InterIIT-TM14.0/raw/main/Task3_Demo_Video.MP4" type="video/mp4">
  Your browser does not support the video tag.
</video>
</div>

---

## About

Adobe Spatia is an AI-powered image editing application that combines fast on-device processing with cloud-assisted generation for high-quality results. It supports features such as colorization, depth-aware editing, object segmentation, scene transfer, and intelligent image analysis. Built with Flutter and ONNX Runtime, it delivers strong performance while preserving user privacy.

---

## Future Of Adobe



https://github.com/user-attachments/assets/25846f81-9931-494e-b554-ab1f03a10732


---

## Current Implemented Apps Results

### Spatial Edit

#### Tunnel Lighting

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/tunnel.png" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/tunnel.png" width="400"/></td> </tr> </table>

#### Cone Lighting

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/cone_lighting.png" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/cone_lighting.png" width="400"/></td> </tr> </table>

#### Area Spotlighting

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/spotlight.png" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/spotlight.png" width="400"/></td> </tr> </table>

#### Zone Lighting

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/zone_lighting.jpg" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/zone_lighting.jpg" width="400"/></td> </tr> </table>

#### Move (Drag & Drop)

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/move.jpg" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/move.jpg" width="400"/></td> </tr> </table>

#### Remove (Erase)

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/remove.png" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/remove.png" width="400"/></td> </tr> </table>

---

### Style Studio

#### Scene Transfer

##### Text-Based

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/style_transfer.png" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/style_transfer.png" width="400"/></td> </tr> </table>

##### Reference Image

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/image_to_image.png" width="400"/></td> <td><strong>Style</strong><br><img src="docs/before/image_to_image_reference.png" width="400"/></td><td><strong>After</strong><br><img src="docs/after/image_to_image.png" width="400"/></td> </tr> </table>

#### Recolor

<table> <tr> <td><strong>Before</strong><br><img src="docs/before/recolor.png" width="400"/></td> <td><strong>After</strong><br><img src="docs/after/recolor.png" width="400"/></td> </tr> </table>

---

## Task 1 — Future Workflow + Speculative Design

Deliverables include an interactive video and a detailed design document illustrating the 2030 vision of Adobe Spatia.

**Includes:**

* PDF and Interactive workflow video containing:

  * Visual storytelling of the 2030 workflow
  * Prototype journeys
  * User personas
  * Decision logs
  * Speculative UI/UX motion studies
  * Screen mockups
  * Design rationale

**Files:**

* `Task 1/Task_1_Interactive_Demo.mp4`
* PDF inside `Task 1/` 

---

## Task 2 — Market Research

A concise two-page market research and Appendix highlighting industry trends, user needs, and opportunity areas.

**Files:**

* `Task 2/`
`2 Page Market Research.pdf`
`Appendix.pdf`


---

## Task 3 — Technical Implementation

Task 3 includes the full technical build of Adobe Spatia, including the mobile app, server components, and a demo video.

### Contents

**1. `product_app/`**
Flutter-based mobile application containing:

* Full cross-platform Flutter code
* Assets, UI, workflows, and environment configuration
* Platform folders (Android, iOS, Web, Desktop)

**2. `server_codebase/`**
Backend and cloud-model setup required for the app, including:

* Model inference pipelines
* API server modules
* Docker setup
* Custom pipelines (SAM, inpainting, etc.)
* Configuration and utilities

**3. Demo Video**

* `Task_3_Demo_Video.MP4`

---

## Repository Structure

```txt
├── Task 1
│   └── Task_1_Interactive_Demo.mp4
│   └── Creative Report.pdf
├── Task 2
│   └── 2 Page Market Research.pdf
├── product_app/          # Flutter application
├── server_codebase/      # Backend & cloud model server
├── optimization/         # Model optimization scripts
├── docs/                 # Additional technical documents
└── Task_3_Demo_Video.MP4
```

---

## Features

### Models & AI Components

Our application integrates multiple state-of-the-art AI models to deliver comprehensive image editing capabilities. The system includes DDColor for image colorization, Depth-Anything-V2 for depth estimation, MobileSAM for object segmentation, LaMa-Dilated for inpainting, CosXL for scene transfer, and MoonDream-v2 for image captioning. Each model is optimized for either on-device or cloud execution based on computational requirements.

**[View Detailed Documentation →](docs/models.md)**

---

### Compute Architecture

The application employs two distinct pipelines: Spatial Editing for lightweight on-device processing and Style Studio for high-fidelity cloud-assisted generation. The compute profile analysis covers model sizes, parameter counts, FLOPs estimation, and deployment strategies. Our architecture is designed to balance performance, memory footprint, and latency across different hardware configurations.

**[View Detailed Documentation →](docs/compute-profile.md)**

---

### Optimization Techniques

To achieve optimal performance on mobile devices, we've implemented various optimization strategies including quantization, pruning, and model compression. The Spatial Lighting pipeline maintains sub-10-second latency with a footprint under 350MB, while the COSXL Edit pipeline has been optimized to reduce GPU memory usage by ~50% through selective quantization.

**[View Detailed Documentation →](docs/optimization.md)**

---

### Training & Fine-tuning

We've fine-tuned SmolVLM-500M using LoRA (Low-Rank Adaptation) for parameter-efficient training focused on understanding weather, lighting, style, and ambience variations in images. This enables sophisticated scene transfer capabilities where the model can extract semantic information from reference images and apply it to target images.

**[View Detailed Documentation →](docs/finetuning.md)**

---

### Privacy & Security

User privacy is a core priority. We've designed a comprehensive privacy architecture that includes confidential computing for cloud-based inference, ensuring that user data remains encrypted and isolated even during processing. The system implements zero-trust principles with hardware-based trusted execution environments (TEEs) for maximum security.

**[View Detailed Documentation →](docs/privacy-confidential-inference.md)**

---

### Ethics & Safety

Our application includes robust guard rails to prevent harmful or inappropriate content generation. We use Gemini API for content safety classification across 25 categories. Additionally, we've integrated blind watermarking technology using DWT-DCT-SVD algorithms to protect digital assets while maintaining visual quality.

**[View Detailed Documentation →](docs/ethics.md)**

---

### Performance Benchmarks

We've evaluated our style transfer pipeline against standard benchmarks, achieving a win rate of ~61.5% against ground truth stylizations. The evaluation uses automated assessment with Gemini 2.0 Flash, comparing generated outputs against reference implementations to ensure quality.

**[View Detailed Documentation →](docs/benchmarking.md)**

---

### Datasets

Our training data combines multiple Kaggle datasets focused on landscape imagery, day/night scenes, and weather conditions. We've created a curated dataset of ~5,700 images with carefully generated captions focusing on weather, ambience, and lighting characteristics using Gemini-2.5-flash-lite.

**[View Detailed Documentation →](docs/datasets.md)**

---

### Future Roadmap

Our hardware roadmap projects full on-device capability for all pipelines within 3-5 years through advances in semiconductor technology (2nm nodes), neural processing units (200+ TOPS), memory architecture (LPDDR6), and storage velocity (UFS 5.0). This evolution will enable real-time high-fidelity generation entirely on mobile devices.

**[View Detailed Documentation →](docs/hardware-roadmap.md)**

---

## Documentation Structure

All detailed documentation is organized in the [`docs/`](docs/) directory:

- **[Models](docs/models.md)** - AI models and their implementations
- **[Compute Profile](docs/compute-profile.md)** - Architecture and performance analysis
- **[Optimization](docs/optimization.md)** - Performance optimization techniques
- **[Fine-tuning](docs/finetuning.md)** - Model training and adaptation
- **[Privacy](docs/privacy-confidential-inference.md)** - Security and privacy architecture
- **[Ethics](docs/ethics.md)** - Safety measures and content protection
- **[Benchmarking](docs/benchmarking.md)** - Performance evaluation results
- **[Datasets](docs/datasets.md)** - Training data and preparation
- **[Hardware Roadmap](docs/hardware-roadmap.md)** - Future technology projections
-  **[Server Side Code Setup](docs/server.md)** - Setup Instruction for Server 

---

Here’s a revised version with the APK link mentioned first, followed by installation steps:

---

## Quick Start

### Download APK

A pre-built release APK is available for quick testing.
[Download here](https://drive.google.com/drive/folders/1RGaBylxQe7OaQG2PAiCq_mrSOVV8CPQ9?usp=sharing)


### Installation (If you want to build locally)

To build and run the app from source, ensure the following prerequisites:

**Prerequisites**

* Flutter SDK 3.9.2 or higher
* Android Studio / Xcode (for mobile development)
* ONNX Runtime for on-device inference
* API keys for cloud services (Gemini, etc.)

**Steps**

1. Clone the repository
2. Navigate to the `product_app/` directory
3. Install dependencies: `flutter pub get`
4. Configure environment variables (see `product_app/.env.example`)
5. Run the app using `flutter run`

For detailed setup instructions, refer to the [App Docs](product_app/Readme.md).

--- 
## License & Acknowledgments

This project integrates multiple open-source models and libraries. Each component retains its original license:

- **DDColor** - Apache 2.0 License
- **Depth-Anything-V2** - Apache 2.0 License (Small model)
- **MobileSAM** - Apache 2.0 License
- **LaMa** - Apache 2.0 License
- **CosXL** - Stability AI Non-Commercial Research Community License
- **MoonDream-v2** - Apache 2.0 License

Please refer to individual documentation files for specific citations and acknowledgments.

---

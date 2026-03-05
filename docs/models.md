# Models

This document details all the AI models used in our application, their general purpose, our specific use cases, and licensing information.

---

## 1. DDCOLOR

### General Purpose

- DDColor can provide vivid and natural colorization for historical black and white old photos
- It can even colorize/recolor landscapes from anime games, transforming your animated scenery into a realistic real-life style! (Image source: Genshin Impact)

### Our Use

We use DDColor for colorizing black and white images in our Magic Transfer feature. The model runs on-device using ONNX Runtime in a background isolate for non-blocking processing. Users can colorize entire images or selectively colorize specific regions by providing masks (created using MobileSAM). The colorization service processes images at 256x256 resolution internally and then resizes the colorized output back to the original image dimensions, combining the colorized AB channels with the original L channel in LAB color space for natural results.

### Acknowledgments / Licenses

This project uses **DDColor**, which is licensed under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).

- **Authors:** Xiaoyang Kang, Tao Yang, Wenqi Ouyang, Peiran Ren, Lingzhi Li, Xuansong Xie.

If you find this work useful, please cite the following paper:

```bibtex
@inproceedings{kang2023ddcolor,
  title={DDColor: Towards Photo-Realistic Image Colorization via Dual Decoders},
  author={Kang, Xiaoyang and Yang, Tao and Ouyang, Wenqi and Ren, Peiran and Li, Lingzhi and Xie, Xuansong},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  pages={328--338},
  year={2023}
}
```

---

## 2. Depth-Anything-V2-Small

### General Purpose

Depth Anything, a highly practical solution for robust monocular depth estimation by training on a combination of 1.5M labeled images and 62M+ unlabeled images. Depth Anything-V2 significantly outperforms V1 in fine-grained details and robustness. Compared with SD-based models, it enjoys faster inference speed, fewer parameters, and higher depth accuracy.

### Our Use

We use Depth-Anything-V2-Small for monocular depth estimation in our Spatial Edit feature. The model runs on-device using ONNX Runtime and processes images at 518x518 resolution. The generated depth maps are cached and used to enable depth-aware spatial lighting effects including Tunnel Lighting, Cone Spotlight, Area Spotlighting, and Zone Lighting. Depth maps are also used for depth-aware object movement, where objects are automatically scaled based on depth differences when moved to different positions in the image. The depth estimation runs in background isolates for optimal performance, and depth maps are resized to match the original image dimensions using bilinear interpolation.

### Licenses and Acknowledgments

- **Code License:** Licensed under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0)
- **Model Weights:**
  - The *Small* model weights are licensed under **Apache 2.0**.
  - The *Base/Large/Giant* model weights are licensed under **CC-BY-NC-4.0** (Non-Commercial).

We strictly adhere to these licenses. The original copyright belongs to the authors at the University of Hong Kong and TikTok.

---

## 3. MobileSAM

### General Purpose

Segment anything model (SAM) is a prompt-guided vision foundation model for cutting out the object of interest from its background. SAM has attracted significant attention due to its impressive zero-shot transfer performance and high versatility of being compatible with other models for advanced vision applications like image editing with fine-grained control. MobileSAM is especially useful for resource constraint edge devices like mobile apps etc.

### Our Use

We use MobileSAM for interactive object segmentation in our app. When users tap on an image, MobileSAM creates precise masks identifying the selected object. The model uses a two-stage architecture with an image encoder and mask decoder, running on-device using ONNX Runtime. We've optimized performance using a persistent isolate pattern that caches image embeddings, reducing tap-to-mask latency from ~9000ms to <200ms. The masks created by MobileSAM are used for multiple features: object removal (with Lama-Dilated), object movement, selective colorization (with DDColor), and scene transfer (with CosXL). The segmentation service initializes a session with the image once and reuses cached embeddings for subsequent segmentation requests.

### Licenses and Acknowledgments

Licensed under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).

- **Authors:** Chaoning Zhang, Dongshen Han, Yu Qiao, Jung Uk Kim, Sung-Ho Bae, Seungkyu Lee, Choong Seon Hong.

```bibtex
@article{mobile_sam,
  title={Faster Segment Anything: Towards Lightweight SAM for Mobile Applications},
  author={Zhang, Chaoning and Han, Dongshen and Qiao, Yu and Kim, Jung Uk and Bae, Sung-Ho and Lee, Seungkyu and Hong, Choong Seon},
  journal={arXiv preprint arXiv:2306.14289},
  year={2023}
}
```

---

## 4. Lama-Dilated

### General Purpose

LaMa-Dilated is a machine learning model that allows to erase and in-paint part of given input image. LaMa is based on i) a new inpainting network architecture that uses fast Fourier convolutions (FFCs), which have the image-wide receptive field; ii) a high receptive field perceptual loss; iii) large training masks, which unlocks the potential of the first two components.

### Our Use

We use LaMa-Dilated for object removal and inpainting in our Spatial Edit feature. When users create a mask using MobileSAM to select an object, LaMa-Dilated fills in the removed area with contextually appropriate content. The model runs on a server via the `/remove/direct` API endpoint, receiving the image and mask as base64-encoded data. LaMa-Dilated is also used in our object movement feature, where it removes objects from their original positions before they are moved to new locations. The model handles large masks effectively and produces high-quality inpainting results that blend seamlessly with the surrounding image content.

### Licenses

- The original LaMa implementation is licensed under the **Apache 2.0 License**.
- The compiled assets from Qualcomm AI Hub are licensed under the **Qualcomm AI Hub Proprietary License**.

### Citation

```bibtex
@article{suvorov2022resolution,
  title={Resolution-robust Large Mask Inpainting with Fourier Convolutions},
  author={Suvorov, Roman and Logacheva, Elizaveta and Mashikhin, Anton and Remizova, Anastasia and Ashukha, Arsenii and Silvestrov, Aleksei and Kong, Naejin and Goka, Harshith and Park, Kiwoong and Lempitsky, Victor},
  journal={arXiv preprint arXiv:2109.07161},
  year={2021}
}
```

---

## 5. CosXL-EDIT 

### User Instructions : 

Download models from these links : 

madebyollin/sdxl-vae-fp16-fix

stabilityai/cosxl

### General Purpose

Cos Stable Diffusion XL 1.0 Base is tuned to use a Cosine-Continuous EDM VPred schedule. The most notable feature of this schedule change is its capacity to produce the full color range from pitch black to pure white, alongside more subtle improvements to the model's rate-of-change to images across each step.

Edit Stable Diffusion XL 1.0 Base is tuned to use a Cosine-Continuous EDM VPred schedule, and then upgraded to perform instructed image editing. This model takes a source image as input alongside a prompt, and interprets the prompt as an instruction for how to alter the image.

### Our Use

We use CosXL (Cos Stable Diffusion XL) for scene transfer and image editing in our Magic Transfer feature. The model runs on a server via the `/cosxl_edit` API endpoint and performs instructed image editing based on text prompts. Users can provide prompts like "make it a cloudy day" or "change to sunset lighting" to transform the scene of their images. CosXL supports optional masks for selective editing, allowing users to apply scene changes only to specific regions identified by MobileSAM. The service automatically resizes and compresses images before sending them to the API to ensure compatibility with server size limits, and handles the full workflow from image preparation to result processing.

### License

CosXL is released under the Stability AI Non-Commercial Research Community License.

**Allowed:** You can use it freely for academic research and non-commercial purposes.

---

## 6. SmolVLM-500M (HuggingFaceTB/SmolVLM-500M-Instruct)

### General Purpose

SmolVLM-500M is a small, efficient vision-language model designed to run on a wide variety of devices (including edge devices like laptops and mobile phones). Its primary capabilities include:

- **Image Captioning:** It can generate short or detailed descriptions of images.
- **Visual Question Answering (VQA):** You can ask it natural language questions about the content of an image (e.g., "How many people are in this image?", "What color is the car?").
- **Object Detection:** It can locate specific objects within an image and provide bounding box coordinates.
- **Pointing:** It can identify the location of objects by providing point coordinates.
- **Grounded Reasoning:** The latest versions include capabilities to "ground" its reasoning in specific spatial positions (like accurately counting or calculating from charts).

It is specifically built to be lightweight and fast compared to massive multimodal models, making it ideal for applications where resources are limited but visual understanding is needed.

### Our Use

We use SmolVLM-500M for image captioning and text extraction in our app. The model runs on a server via the Text Extractor API (`/caption/ambience` endpoint) and is primarily used to extract ambience, weather, lighting, and season descriptions from reference images. These extracted captions are then used as prompts for the CosXL scene transfer feature, allowing users to automatically transfer the mood and atmosphere from one image to another. MoonDream-v2's lightweight architecture makes it ideal for server-side processing, and its vision-language capabilities enable natural language descriptions that work seamlessly with our text-based editing workflows.

### License

The model is released under the Apache 2.0 License.

**Permitted:** Commercial use, modification, distribution, and private use are allowed.


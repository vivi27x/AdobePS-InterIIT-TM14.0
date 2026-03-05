# User Data Privacy & Confidential Inference

To address strict privacy requirements regarding user data, we design a future plan for this project. We project that mobile NPU evolution (e.g., Snapdragon 8 Elite successors) will eventually run our current CosXL pipeline entirely locally, ensuring data never physically leaves the device. However, to provide **state-of-the-art Generative AI** eg. Nano Banana Pro-32B parameters (and to support future "Ultra-scale" models that exceed mobile capabilities), we utilize a hybrid architecture. To satisfy the strict requirement that user data must remain private—as if it never left the device—we implement a **Zero-Trust Confidential Inference System**.

## 1. The "Virtual On-Device" Guarantee

We bridge the gap between current hardware constraints and privacy requirements using **Confidential Computing**. This technology ensures that when we offload heavy tasks (like 34B+ parameter Image generation models) to the cloud, the data remains cryptographically isolated.

In this architecture, the cloud provider (AWS/GCP/Azure) is treated as an **untrusted entity**. User photos and prompts are processed inside a hardware-based **Trusted Execution Environment (TEE)**. This guarantees that:

- **Data is invisible to the Cloud Provider:** The infrastructure owner cannot view the memory contents.
- **Data is invisible to the OS:** The host operating system and hypervisor cannot access the application state.
- **Data is used for Inference Only:** Mathematical guarantees ensure data exists in plaintext *only* during the split-second of computation within the CPU/GPU die, and is immediately wiped thereafter.

## 2. Enabling "Ultra" Models with Confidential GPUs

While our roadmap brings CosXL to mobile devices, our architecture is designed to scale to **frontier models (100B+ parameters)** that will always require server-grade hardware.

Standard CPU-based confidential computing (e.g., Intel TDX) is insufficient for these generative workloads due to high latency. Therefore, our architecture specifies the use of **NVIDIA H100 Tensor Core GPUs** with Confidential Computing support.

- **Why H100?** It extends the TEE boundary from the CPU to the GPU, encrypting transfers over the PCIe bus.
- **Performance:** It allows us to run heavy generative pipelines at full speed without compromising the "Black Box" privacy guarantee.

This ensures that while our standard pipeline migrates to mobile, this secure cloud architecture remains vital for running next-generation "Pro" features without privacy trade-offs.

## 3. The Zero-Trust Data Lifecycle

This workflow ensures that from the perspective of the network and the cloud admin, the data is never revealed.

1. **Local Encryption:** The app generates a session key ($K_{sess}$) and encrypts the image locally.
2. **Remote Attestation:** The app challenges the server to prove it is running inside a genuine TEE with the correct, un-tampered AI model code.
3. **Secure Handshake:** Only upon verifying the hardware signature (Attestation Report), the app transmits the key directly to the secure enclave.
4. **Isolated Inference:**
   - The encrypted payload moves directly to the **Confidential GPU** memory.
   - Inference occurs in isolated memory regions protected from the host.
   - Input data and intermediate tensors are effectively "invisible" to the outside world.
5. **Destruction & Return:** The output is encrypted inside the enclave, returned to the user, and all traces are securely erased from the server memory.

---


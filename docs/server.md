# Docker + GPU + tmux Setup

This guide explains how to containerize and run the Techmeet Pipeline using Docker with:

- **NVIDIA GPU Support**
- **Host Networking** (no port mapping needed)
- **tmux-based multi-service startup**
- **Hot-reload using volume mounts**

---

## 1. Prerequisites

Before running the container, ensure your host system has:

### Docker Engine
Install Docker if you haven't already:
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### NVIDIA Drivers
Check with:
```bash
nvidia-smi
```

### NVIDIA Container Toolkit
Required for Docker to access the GPU.

#### Install NVIDIA Container Toolkit (Ubuntu / Debian)

If you haven't installed it yet:

```bash
# Add the repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install and configure
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

---

## 2. Download Model Weights

Before building the Docker image, you need to download the required model weights.

### A. Big-LAMA Weights

Download the Big-LAMA model weights and place them in the `big-lama/` folder.


**Instructions:**
1. Download the weights from the provided [Google Drive link](https://drive.google.com/drive/folders/1wHtkLBcPmHKcFJz1dkGXW01OTSrKr2Fh?usp=sharing)
2. Extract the downloaded files (if compressed)
3. Place all weight files in the `big-lama/` directory

**Expected structure:**
```
big-lama/
├── config.yaml
├── models/
│   ├── best.ckpt
│   ├── generator_compressed.pt
│   └── generator_pruned.pt
└── [other weight files from download]
```

### B. MobileSAM and CoSXL Weights

Download the MobileSAM and CoSXL model weights and place them in the `models/` folder.

**Instructions:**
1. Download the weights from the provided [Google Drive link](https://drive.google.com/drive/folders/1wHtkLBcPmHKcFJz1dkGXW01OTSrKr2Fh?usp=sharing)
2. Extract the downloaded files (if compressed)
3. Place the weight files in the `models/` directory

**Expected structure:**
```
models/
├── mobile_sam.pt          # MobileSAM weights
├── cosxl_edit.safetensors # CoSXL weights
└──[other model files]
```

**Note:** Make sure the weight files are in place before building the Docker image, as they will be included in the container via volume mounting.

---

## 3. Project File Structure

Ensure this structure exists in your project root:

```
.
├── Dockerfile
├── entrypoint.sh
├── nginx.conf
├── requirements.txt
├── big-lama/
│   └── [weight files]
├── models/
│   └── [weight files]
├── mainserver.py
└── run_server.py
```

### A. Dockerfile

The Dockerfile should contain:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    tmux \
    ffmpeg \
    libsm6 \
    libxext6 \
    nginx \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install
COPY requirements.txt .
RUN pip install --upgrade pip
RUN pip install --no-cache-dir --default-timeout=1000 -r requirements.txt

# Copy Nginx configuration
COPY nginx.conf /etc/nginx/sites-available/default

# Environment variable placeholder
ENV GEMINI_API_KEY=""

# Entrypoint (mounted via volume)
CMD ["./entrypoint.sh"]
```

### B. entrypoint.sh

Make this file executable:

```bash
chmod +x entrypoint.sh
```

**Contents:**

```bash
#!/bin/bash

# Start Nginx in the background
nginx

# Start tmux session for running both services
tmux new-session -d -s techmeet_session 'python3 mainserver.py'

# Split horizontally and run second script
tmux split-window -h -t techmeet_session 'python3 run_server.py'

# Attach to session
tmux attach-session -t techmeet_session
```

### C. nginx.conf

Nginx configuration file for routing:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /feature/ {
        proxy_pass http://127.0.0.1:8001/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location = /nginx-health {
        return 200 "nginx OK\n";
        add_header Content-Type text/plain;
    }
}
```
```
sudo nginx -t
sudo systemctl restart nginx
```
**Routing:**
- Port 80 (root `/`) → Proxies to `http://127.0.0.1:8000` (mainserver.py)
- Port 80 (`/feature/`) → Proxies to `http://127.0.0.1:8001/` (run_server.py)
- Health check endpoint: `/nginx-health`

---

## 4. Build the Docker Image

Run from project root:

```bash
docker build -t techmeet-app .
```

---

##  5. Run the Container

This command enables GPU, host networking, live code editing, and environment variables:

```bash
docker run --gpus all --privileged --net=host -it --rm \
  -v $(pwd):/app \
  -e GEMINI_API_KEY="YOUR_ACTUAL_API_KEY_HERE" \
  techmeet-app
```

###  Explanation of Flags

| Flag | Meaning |
|------|---------|
| `--gpus all` | Enables full NVIDIA GPU access |
| `--privileged` | Fix for NVML + permission issues |
| `--net=host` | Exposes services directly on localhost (80 via Nginx, 8000, 8001) |
| `-it` | Interactive mode (needed for tmux) |
| `--rm` | Deletes container after exit |
| `-v $(pwd):/app` | Mounts project directory for live code updates |
| `-e GEMINI_API_KEY=` | Pass your API key into container |

---

## 6. Using tmux Inside the Container

Once the container is running, you'll be inside a tmux session with two panes:

- **Left pane:** Running `mainserver.py`
- **Right pane:** Running `run_server.py`

### tmux Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+B` then `D` | Detach from session (keeps container running) |
| `Ctrl+B` then `%` | Split pane vertically |
| `Ctrl+B` then `"` | Split pane horizontally |
| `Ctrl+B` then arrow keys | Navigate between panes |
| `Ctrl+B` then `X` | Close current pane |
| `Ctrl+B` then `C` | Create new window |

**To reattach to the session:**
```bash
# From inside the container
tmux attach-session -t techmeet_session

# Or from host (if container is running)
docker exec -it <container_id> tmux attach-session -t techmeet_session
```

---

## 7. Nginx Reverse Proxy

The container includes Nginx as a reverse proxy that routes traffic:

- **Port 80 (root `/`)** → Routes to `http://127.0.0.1:8000` (mainserver.py)
- **Port 80 (`/feature/`)** → Routes to `http://127.0.0.1:8001/` (run_server.py)
- **Health Check:** `http://localhost/nginx-health` returns "nginx OK"

### Accessing Services

Once the container is running:

- Main server: `http://localhost/` or `http://localhost:8000/`
- Feature server: `http://localhost/feature/` or `http://localhost:8001/`
- Nginx health: `http://localhost/nginx-health`

### Nginx Management

**Check Nginx status:**
```bash
# From inside the container
docker exec -it <container_id> nginx -t  # Test configuration
docker exec -it <container_id> nginx -s reload  # Reload configuration
```

**View Nginx logs:**
```bash
docker exec -it <container_id> tail -f /var/log/nginx/error.log
docker exec -it <container_id> tail -f /var/log/nginx/access.log
```

---



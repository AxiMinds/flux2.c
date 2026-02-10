# ComfyUI + A1111 Combined Container with CUDA
# For dual NVIDIA RTX 5060 Ti GPUs (Blackwell architecture)
# Uses PyTorch nightly with CUDA 13.0 for optimized Blackwell operations

FROM nvidia/cuda:12.8.0-cudnn-runtime-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV UV_SYSTEM_PYTHON=1

# A1111 repository overrides (Stability-AI repos are now private)
ENV STABLE_DIFFUSION_REPO=https://github.com/w-e-w/stablediffusion.git
ENV STABLE_DIFFUSION_XL_REPO=https://github.com/w-e-w/generative-models.git

# Force GPU-only PyTorch operation
ENV CUDA_LAUNCH_BLOCKING=0
ENV TORCH_CUDA_ARCH_LIST="8.0;8.6;8.9;9.0;12.0"
ENV FORCE_CUDA=1

# System dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    python3.11 \
    python3.11-venv \
    python3-pip \
    wget \
    curl \
    libgl1 \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender1 \
    libgoogle-perftools4 \
    bc \
    && rm -rf /var/lib/apt/lists/*

# Set Python 3.11 as default
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1 \
    && update-alternatives --install /usr/bin/python python /usr/bin/python3.11 1

# Install uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:$PATH"

WORKDIR /app

# Install PyTorch nightly with CUDA 13.0 for Blackwell (RTX 50xx) optimized operations
RUN uv pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu130

# Clone ComfyUI
RUN git clone https://github.com/comfyanonymous/ComfyUI.git /app/comfyui && \
    cd /app/comfyui && \
    uv pip install -r requirements.txt

# Clone A1111 Stable Diffusion WebUI
RUN git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git /app/a1111

# Pre-clone A1111's required repositories (uses community forks since Stability-AI repos are now private)
# See: https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/17212
RUN mkdir -p /app/a1111/repositories && \
    git clone https://github.com/w-e-w/stablediffusion.git /app/a1111/repositories/stable-diffusion-stability-ai && \
    git clone https://github.com/w-e-w/generative-models.git /app/a1111/repositories/generative-models && \
    git clone https://github.com/crowsonkb/k-diffusion.git /app/a1111/repositories/k-diffusion && \
    git clone https://github.com/salesforce/BLIP.git /app/a1111/repositories/BLIP

# Install A1111 dependencies
RUN cd /app/a1111 && \
    uv pip install -r requirements_versions.txt || true

# Common dependencies for FLUX models - force upgrade transformers for ComfyUI Qwen2Tokenizer support
RUN uv pip install \
    accelerate \
    "transformers>=4.45.0" \
    safetensors \
    diffusers \
    sentencepiece \
    protobuf \
    "pydantic>=2.0"

# Reinstall ComfyUI requirements to ensure compatibility (after A1111 downgrades)
RUN cd /app/comfyui && uv pip install -r requirements.txt

# Create shared directories
RUN mkdir -p /app/models/checkpoints \
    /app/models/vae \
    /app/models/loras \
    /app/models/controlnet \
    /app/models/clip \
    /app/models/unet \
    /app/outputs/comfyui \
    /app/outputs/a1111

# Install ComfyUI Manager
RUN cd /app/comfyui/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Manager.git

# Install FLUX-specific ComfyUI nodes
RUN cd /app/comfyui/custom_nodes && \
    git clone https://github.com/city96/ComfyUI-GGUF.git && \
    git clone https://github.com/kijai/ComfyUI-Florence2.git && \
    uv pip install -r ComfyUI-GGUF/requirements.txt || true && \
    uv pip install -r ComfyUI-Florence2/requirements.txt || true

# Dependencies for ComfyUI-RMBG background removal node (custom_nodes volume-mounted)
RUN uv pip install transparent_background

# Symlink shared models to both apps
RUN ln -sf /app/models/checkpoints /app/comfyui/models/checkpoints && \
    ln -sf /app/models/vae /app/comfyui/models/vae && \
    ln -sf /app/models/loras /app/comfyui/models/loras && \
    ln -sf /app/models/controlnet /app/comfyui/models/controlnet && \
    ln -sf /app/models/clip /app/comfyui/models/clip && \
    ln -sf /app/models/unet /app/comfyui/models/unet

RUN mkdir -p /app/a1111/models/Stable-diffusion && \
    ln -sf /app/models/checkpoints/* /app/a1111/models/Stable-diffusion/ 2>/dev/null || true && \
    ln -sf /app/models/vae /app/a1111/models/VAE && \
    ln -sf /app/models/loras /app/a1111/models/Lora

# Startup script
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh

EXPOSE 9101 9102

CMD ["/app/start.sh"]

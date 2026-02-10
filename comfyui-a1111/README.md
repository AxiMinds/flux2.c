# ComfyUI + Forge Docker Stack

Three-service stack: ComfyUI (FLUX + SDXL), Forge (SDXL), and ImageGen Gallery with AI-powered image editor. Dual NVIDIA RTX 5060 Ti (Blackwell) GPUs with CUDA via PyTorch nightly.

## Services

| Service | Port | Container | GPU | Purpose |
|---------|------|-----------|-----|---------|
| **ComfyUI** | 9101 | `comfyui` | 0+1 | FLUX.1-dev/schnell, SDXL workflows, upscale, segmentation, bg removal |
| **Forge** | 9102 | `forge` | 0+1 | SDXL txt2img/img2img REST API |
| **Gallery** | 9201 | `imagegen-gallery` | — | Web UI: image browser, AI chat, image editor |

## Quick Start

```bash
docker-compose up -d

# Access:
# ComfyUI:  http://172.30.200.200:9101
# Forge:    http://172.30.200.200:9102
# Gallery:  http://172.30.200.200:9201
```

## Image Editor (Gallery)

The gallery includes a full image editor accessible by clicking any image → "Open in Editor". Two modes:

**Manual Mode** — Direct control over all parameters:
- img2img (Forge SDXL or ComfyUI SDXL)
- Outpaint with directional padding (ComfyUI SDXL)
- Neural upscale (RealESRGAN, UltraSharp) or Lanczos
- Object segmentation with text prompt (INSPYRENET)
- Background removal (INSPYRENET)
- txt2img (all backends: Forge SDXL, ComfyUI SDXL, FLUX dev/schnell)

**AI Mode** — Natural language instructions processed by Ollama:
- Describe edits in plain English → AI selects operation, backend, and parameters
- AI rewrites prompts (shown for learning)
- Optional quality check with auto-retry
- Character preservation for consistent likeness
- SSE-streamed progress

**Interactive Demo** — Click "Demo" button for a guided 3-phase walkthrough (txt2img → img2img → AI mode → upscale) using real API calls with typewriter animations and narrated explanations.

## Models

```
models/
├── checkpoints/    sd_xl_base_1.0.safetensors (6.5G)
├── unet/           flux1-dev.safetensors (23G), flux1-schnell.safetensors (23G)
├── clip/           clip_l.safetensors (235M), t5xxl_fp8_e4m3fn.safetensors (4.6G)
├── vae/            flux_ae.safetensors, sdxl_vae.safetensors
├── upscale_models/ RealESRGAN_x4plus.pth, 4x-UltraSharp.pth
├── loras/          Qwen-Image-Edit variants
└── text_encoders/  qwen_2.5_vl_7b_fp8_scaled.safetensors (8.8G)
```

## Custom Nodes (ComfyUI)

| Node | Purpose | Dependencies |
|------|---------|-------------|
| ComfyUI-Manager | Node package manager | — |
| ComfyUI-GGUF | GGUF model loading | gguf |
| ComfyUI-Florence2 | Vision-language | — |
| ComfyUI-RMBG | Background removal & segmentation | `transparent_background` |
| comfyui-inpaint-nodes | Inpainting/outpainting | — |

## Workflow Templates

| Template | Operation | Backend |
|----------|-----------|---------|
| `flux-dev-txt2img.json` | FLUX.1-dev text-to-image | ComfyUI |
| `flux-schnell-txt2img.json` | FLUX.1-schnell text-to-image | ComfyUI |
| `sdxl-txt2img.json` | SDXL text-to-image | ComfyUI |
| `sdxl-img2img.json` | SDXL image-to-image | ComfyUI |
| `outpaint-sdxl.json` | Outpaint with padding | ComfyUI |
| `upscale-model.json` | Neural upscale (ESRGAN) | ComfyUI |
| `upscale-lanczos.json` | Lanczos upscale | ComfyUI |
| `segment-object.json` | Object segmentation | ComfyUI |
| `segment-remove-bg.json` | Background removal | ComfyUI |

## Environment Variables (Gallery)

| Variable | Default | Description |
|----------|---------|-------------|
| `OLLAMA_URL` | `http://172.30.200.200:11434` | Ollama API |
| `OLLAMA_MODEL` | `qwen3-coder:30b` | Chat model |
| `EDITOR_AI_MODEL` | `qwen3:32b` | Editor AI intent model |
| `EDITOR_QUALITY_MODEL` | `qwen3:8b` | Editor quality check model |
| `PROMPT_REFACTOR_MODEL` | `qwen3:32b` | Prompt rewriting |
| `CONSCIOUSNESS_MODELS` | `qwen3-coder:30b` | Persona inner/outer models |
| `EMOTION_MODEL` | `qwen3:0.6b` | Emotion engine model |

## Rebuilding

```bash
# Rebuild ComfyUI (after Dockerfile changes)
docker-compose build comfyui && docker-compose up -d comfyui

# Rebuild Forge
docker-compose build forge && docker-compose up -d forge

# Gallery uses python:3.12-slim directly (no build needed)
docker-compose restart gallery
```

## VRAM Usage

Each RTX 5060 Ti has 16GB VRAM:

| Model | VRAM |
|-------|------|
| FLUX.1-dev | ~12GB |
| FLUX.1-schnell | ~10GB |
| SDXL 1.0 | ~8GB |
| RealESRGAN upscale | ~2GB |
| INSPYRENET segmentation | ~1GB |

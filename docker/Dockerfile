# syntax=docker/dockerfile:1
# 3D Gaussian Splatting — pip + CUDA 12.8 (Blackwell sm_120 compatible)
# Build from repo root:
#   docker compose -f docker/docker-compose.yml -f docker/docker-compose.gpu.yml build

FROM pytorch/pytorch:2.7.1-cuda12.8-cudnn9-devel

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV CUDA_HOME=/usr/local/cuda
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64:${LD_LIBRARY_PATH}
# RTX PRO 6000 Blackwell (compute capability 12.0)
ENV TORCH_CUDA_ARCH_LIST="12.0+PTX"
ENV FORCE_CUDA=1
ENV PYTHONUNBUFFERED=1

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    rm -f /etc/apt/apt.conf.d/docker-clean \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        curl \
        git \
        libgl1-mesa-glx \
        libglib2.0-0 \
        python3 \
        python3-dev \
        python3-pip \
        wget \
    && ln -sf /usr/bin/python3 /usr/bin/python

RUN --mount=type=cache,target=/root/.cache/pip \
    python -m pip install --upgrade pip setuptools wheel

# PyTorch >= 2.7 + cu128 required for Blackwell (sm_120).
# The cu128 stack pulls ~3-4 GB of CUDA wheels over a flaky link. Two mechanisms
# PyTorch / torchvision / torchaudio are provided by the base image:
#   pytorch/pytorch:2.7.1-cuda12.8-cudnn9-devel
# Do not pip-install torch here, otherwise Docker build may stall on large NVIDIA wheels.

WORKDIR /workspace/gaussian-splatting

# Copy source + submodules for CUDA extension build
COPY . /workspace/gaussian-splatting

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        plyfile \
        tqdm \
        opencv-python \
        joblib \
    && pip install --no-build-isolation submodules/diff-gaussian-rasterization \
    && pip install --no-build-isolation submodules/simple-knn \
    && pip install --no-build-isolation submodules/fused-ssim

RUN python - <<'PY'
import torch
from diff_gaussian_rasterization import GaussianRasterizer
print("PyTorch:", torch.__version__, "CUDA:", torch.version.cuda)
print("GPU:", torch.cuda.get_device_name(0) if torch.cuda.is_available() else "none (build without GPU is OK)")
print("arch list:", torch.cuda.get_arch_list())
print("GaussianRasterizer import OK")
PY

WORKDIR /workspace/gaussian-splatting
CMD ["bash"]

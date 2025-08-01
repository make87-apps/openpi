# syntax=docker/dockerfile:1.4

FROM nvidia/cuda:12.2.2-cudnn8-runtime-ubuntu22.04@sha256:2d913b09e6be8387e1a10976933642c73c840c0b735f0bf3c28d97fc9bc422e0 AS base

COPY --from=ghcr.io/astral-sh/uv:0.5.1 /uv /uvx /bin/
WORKDIR /app

RUN apt-get update && apt-get install -y \
    git git-lfs curl linux-headers-generic build-essential clang \
    && apt-get clean

ENV UV_LINK_MODE=copy
ENV UV_PROJECT_ENVIRONMENT=/.venv
ENV OPENPI_DATA_HOME=/models
ENV PORT=80

RUN git clone --depth=1 https://github.com/Physical-Intelligence/openpi.git . && \
    uv venv --python 3.11.9 $UV_PROJECT_ENVIRONMENT && \
    GIT_LFS_SKIP_SMUDGE=1 uv sync --frozen --no-install-project --no-dev

# Add a helper script to download checkpoints
COPY <<EOF download_checkpoint.py
import sys
from openpi.shared.download import maybe_download

if len(sys.argv) != 2:
    print("Usage: python download_checkpoint.py <gs://path>")
    sys.exit(1)

maybe_download(sys.argv[1])
EOF

# ========== Target: pi0_aloha ==========
FROM base AS pi0_aloha
ENV OPENPI_DATA_HOME=/models/pi0_aloha
RUN uv run python download_checkpoint.py gs://openpi-assets/checkpoints/pi0_base
ENTRYPOINT ["uv", "run", "scripts/serve_policy.py", "--env", "ALOHA"]

# ========== Target: pi0_fast_droid ==========
FROM base AS pi0_fast_droid
ENV OPENPI_DATA_HOME=/models/pi0_fast_droid
RUN uv run python download_checkpoint.py gs://openpi-assets/checkpoints/pi0_fast_droid
ENTRYPOINT ["uv", "run", "scripts/serve_policy.py", "--env", "DROID"]

# ========== Target: pi0_fast_libero ==========
FROM base AS pi0_fast_libero
ENV OPENPI_DATA_HOME=/models/pi0_fast_libero
RUN uv run python download_checkpoint.py gs://openpi-assets/checkpoints/pi0_fast_libero
ENTRYPOINT ["uv", "run", "scripts/serve_policy.py", "--env", "LIBERO"]

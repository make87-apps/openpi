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

# Clone openpi client and script
# Clone openpi client and script
RUN git clone --depth=1 https://github.com/Physical-Intelligence/openpi.git /tmp/openpi && \
    mkdir -p packages/openpi-client && \
    cp -r /tmp/openpi/packages/openpi-client/* packages/openpi-client/ && \
    cp /tmp/openpi/pyproject.toml ./ && \
    cp /tmp/openpi/uv.lock ./ && \
    mkdir -p scripts && \
    cp /tmp/openpi/scripts/serve_policy.py scripts/ && \
    rm -rf /tmp/openpi

RUN uv venv --python 3.11.9 $UV_PROJECT_ENVIRONMENT && \
    GIT_LFS_SKIP_SMUDGE=1 uv sync --frozen --no-install-project --no-dev

# ========== Target: pi0_fast_droid ==========
FROM base AS pi0_fast_droid
RUN mkdir -p /models/pi0_fast_droid && \
    curl -L https://storage.googleapis.com/openpi-assets/checkpoints/pi0_fast_droid/policy_params.pkl -o /models/pi0_fast_droid/policy_params.pkl && \
    curl -L https://storage.googleapis.com/openpi-assets/checkpoints/pi0_fast_droid/policy_weights.msgpack -o /models/pi0_fast_droid/policy_weights.msgpack
ENTRYPOINT ["uv", "run", "scripts/serve_policy.py", "policy:checkpoint", "--policy.config=pi0_fast_droid", "--policy.dir=/models/pi0_fast_droid"]

# ========== Target: pi0_droid ==========
FROM base AS pi0_droid
RUN mkdir -p /models/pi0_droid && \
    curl -L https://storage.googleapis.com/openpi-assets/checkpoints/pi0_droid/policy_params.pkl -o /models/pi0_droid/policy_params.pkl && \
    curl -L https://storage.googleapis.com/openpi-assets/checkpoints/pi0_droid/policy_weights.msgpack -o /models/pi0_droid/policy_weights.msgpack
ENTRYPOINT ["uv", "run", "scripts/serve_policy.py", "policy:checkpoint", "--policy.config=pi0_droid", "--policy.dir=/models/pi0_droid"]

# ========== Target: pi0_aloha_towel ==========
FROM base AS pi0_aloha_towel
RUN mkdir -p /models/pi0_aloha_towel && \
    curl -L https://storage.googleapis.com/openpi-assets/checkpoints/pi0_aloha_towel/policy_params.pkl -o /models/pi0_aloha_towel/policy_params.pkl && \
    curl -L https://storage.googleapis.com/openpi-assets/checkpoints/pi0_aloha_towel/policy_weights.msgpack -o /models/pi0_aloha_towel/policy_weights.msgpack
ENTRYPOINT ["uv", "run", "scripts/serve_policy.py", "policy:checkpoint", "--policy.config=pi0_aloha_towel", "--policy.dir=/models/pi0_aloha_towel"]

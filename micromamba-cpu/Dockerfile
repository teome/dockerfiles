FROM docker.io/mambaorg/micromamba:1-jammy

LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
    wget bzip2 ca-certificates \
    git \
    htop \
    vim.tiny \
    tmux \
    tini && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN micromamba install --yes --name base \
    --channel pytorch-nightly \
    --channel nvidia \
    --channel conda-forge \
    python=${PYTHON_VERSION} \
    pytorch torchvision torchaudio cpuonly \
    cmake pip pyyaml ipython jupyter black pylint \
    transformers sentencepiece accelerate wandb \
    numpy mkl "libblas=*=*mkl" && \
    micromamba clean --all --yes
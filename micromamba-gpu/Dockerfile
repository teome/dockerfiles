FROM docker.io/mambaorg/micromamba:1-jammy-cuda-11.8.0

LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10
ARG CUDA_VERSION=11.8
ARG MAMBA_USER=mambauser

# mamba container uses non-root user, so change to root for apt
USER root

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

RUN mkdir -p /certs && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /certs/mykey.key \
    -out /certs/mycert.pem \
    -subj "/C=GB/ST=London/L=London/O=Global Security/OU=AGI/CN=jnb.ai" && \
    chmod 755 /certs && \
    chmod 644 /certs/*

USER $MAMBA_USER

# Setup notebook ssl and https
RUN mkdir -p /home/${MAMBA_USER}/.jupyter
COPY --chown=$MAMBA_USER:$MAMBA_USER jupyter_notebook_config.py /home/${MAMBA_USER}/.jupyter/jupyter_notebook_config.py

RUN micromamba install --yes --name base \
    --channel pytorch-nightly \
    --channel nvidia \
    --channel conda-forge \
    python=${PYTHON_VERSION} \
    pytorch torchvision torchaudio pytorch-cuda=${CUDA_VERSION} \
    cmake pip pyyaml ipython jupyter black pylint \
    transformers sentencepiece accelerate wandb \
    numpy mkl "libblas=*=*mkl" && \
    micromamba clean --all --yes

WORKDIR /home/${MAMBA_USER}
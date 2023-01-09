FROM docker.io/mambaorg/micromamba:1-jammy-cuda-11.8.0

LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10
ARG CUDA_VERSION=11.8
ARG MAMBA_USER=mamba

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

USER $MAMBA_USER

# TODO: remove explicit pinned libcufile, should be auto installed
# https://github.com/mamba-org/mamba/issues/2167
RUN micromamba install --yes --name base \
    --channel pytorch-nightly \
    --channel nvidia \
    --channel conda-forge \
    python=${PYTHON_VERSION} \
    pytorch torchvision torchaudio pytorch-cuda=${CUDA_VERSION} \
    libcufile=1.4.0.31 \
    cmake pip pyyaml ipython jupyter black pylint \
    transformers sentencepiece accelerate wandb \
    numpy mkl "libblas=*=*mkl" && \
    micromamba clean --all --yes

# Setup notebook ssl and https
RUN mkdir -p /home/${MAMBA_USER}/.jupyter/ && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /home/${MAMBA_USER}/mykey.key \
    -out /home/${MAMBA_USER}/mycert.pem

COPY <<EOF /home/${MAMBA_USER}/.jupyter/
c.NotebookApp.certfile = u'/home/${MAMBA_USER}/mycert.pem'
c.NotebookApp.keyfile = u'/home/${MAMBA_USER}/mykey.key'
# Set ip to '*' to bind on all interfaces (ips) for the public server
c.NotebookApp.ip = '*'
c.NotebookApp.password = u'sha1:bcd259ccf...<your hashed password here>'
c.NotebookApp.open_browser = False
# It is a good idea to set a known, fixed port for server access
c.NotebookApp.port = 9999
EOF


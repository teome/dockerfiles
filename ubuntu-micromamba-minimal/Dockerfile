ARG BASE_IMAGE=docker.io/ubuntu:22.04
FROM $BASE_IMAGE

LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10
ARG MAMBA_VERSION=latest

ARG USERNAME=dom
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV ENV_NAME="base"
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"
ENV PATH=${MAMBA_ROOT_PREFIX}/bin:${PATH}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
    wget curl bzip2 ca-certificates \
    git \
    htop \
    vim.tiny \
    sudo \
    tmux \
    tini && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

# DL and setup mamba root prefix -- needed as we're using bare micromamba
# TODO fix TARGETARCH spec
ARG TARGETARCH=amd64
RUN test "$TARGETARCH" = 'amd64' && export ARCH='64'; \
    test "$TARGETARCH" = 'arm64' && export ARCH='aarch64'; \
    test "$TARGETARCH" = 'ppc64le' && export ARCH='ppc64le'; \
    curl -L "https://micro.mamba.pm/api/micromamba/linux-${ARCH}/${MAMBA_VERSION}" | \
    tar -xj -C "/" "bin/micromamba" && \
    mkdir -p "$MAMBA_ROOT_PREFIX/conda-meta" && \
    chmod -R a+rwx "$MAMBA_ROOT_PREFIX"

RUN mkdir -p /certs && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /certs/mykey.key \
    -out /certs/mycert.pem \
    -subj "/C=GB/ST=London/L=London/O=Global Security/OU=AGI/CN=jnb.ai" && \
    chmod 755 /certs && \
    chmod 644 /certs/*

USER $USERNAME

# Setup notebook ssl and https
RUN mkdir -p /home/${USERNAME}/.jupyter
COPY --chown=$USERNAME:$USERNAME jupyter_notebook_config.py /home/${USERNAME}/.jupyter/jupyter_notebook_config.py

# TODO: remove explicit pinned libcufile, should be auto installed
# https://github.com/mamba-org/mamba/issues/2167
RUN micromamba install --name base --yes \
    --channel pytorch-nightly \
    --channel conda-forge \
    python=${PYTHON_VERSION} \
    pytorch torchvision torchaudio cpuonly \
    cmake pip pyyaml ipython jupyter black pylint \
    transformers sentencepiece accelerate wandb \
    numpy mkl "libblas=*=*mkl" && \
    micromamba clean --all --yes

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

ARG BASE_IMAGE=docker.io/nvidia/cuda:11.8.0-runtime-ubuntu22.04
FROM $BASE_IMAGE

LABEL maintainer="Dominic Kelly"

ARG DEBIAN_FRONTEND=noninteractive

# TODO: check on progress for pytorch support for python3.11
# When 3.11 is available, change and use deadsnakes ppa
# ARG PYTHON_VERSION=3.11
ARG PYTHON_VERSION=3.10
ARG CUDA_VERSION=11.8

ARG USERNAME=dom
ARG USER_UID=1000
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV ENV_NAME="base"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
    wget curl bzip2 ca-certificates \
    git \
    htop \
    vim.tiny \
    sudo \
    tmux \
    tini \
    graphviz \
    python${PYTHON_VERSION} \
    python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
RUN groupadd --gid ${USER_UID} ${USERNAME} && \
    useradd -m --uid ${USER_UID} --gid ${USER_UID} -s /bin/bash -G sudo ${USERNAME} && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Use the correct python version (taken from mosaicml)
# Set the default python by creating our own folder and hacking the path
# We don't want to use upgrade-alternatives as that will break system packages
ARG PYTHON_BIN=/python-bin

RUN mkdir -p ${PYTHON_BIN} && \
    ln -s $(which python${PYTHON_VERSION}) ${PYTHON_BIN}/python && \
    ln -s $(which python${PYTHON_VERSION}) ${PYTHON_BIN}/python3 && \
    ln -s $(which python${PYTHON_VERSION}) ${PYTHON_BIN}/python${PYTHON_VERSION} && \
    ln -s $(which pip${PYTHON_VERSION}) ${PYTHON_BIN}/pip && \
    ln -s $(which pip${PYTHON_VERSION}) ${PYTHON_BIN}/pip3 && \
    ln -s $(which pip${PYTHON_VERSION}) ${PYTHON_BIN}/pip${PYTHON_VERSION} && \
    # Include this folder, and the local bin folder, on the path
    echo "export PATH=~/.local/bin:$PYTHON_BIN:$PATH" >> /etc/profile && \
    echo "export PATH=~/.local/bin:$PYTHON_BIN:$PATH" >> /etc/bash.bashrc && \
    echo "export PATH=~/.local/bin:$PYTHON_BIN:$PATH" >> /etc/zshenv

# Ensure that non-interactive shells load /etc/profile
ENV BASH_ENV=/etc/profile
# And use this python version for rest of the build
ENV PATH=$PYTHON_BIN:$PATH

RUN python3 -m pip install --upgrade pip && \
    python3 -m pip --no-cache-dir install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 && \
    python3 -m pip install --no-cache-dir \
    cmake pip pyyaml ipython notebook matplotlib black pylint graphviz pytest \
    sentencepiece transformers accelerate diffusers wandb einops && \
    python3 -m pip cache purge

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

ENV PATH=/home/${USERNAME}/.local/bin:$PATH
WORKDIR /home/${USERNAME}

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

ARG BASE_IMAGE=docker.io/nvidia/cuda:11.7.0-runtime-ubuntu22.04
FROM $BASE_IMAGE

LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.11
ARG CUDA_VERSION=11.7

ARG USERNAME=dom
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV ENV_NAME="base"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME  && \
    echo "alias python=python${PYTHON_VERSION}" >> /home/${USERNAME}/.bashrc && \
    echo "alias python3=python${PYTHON_VERSION}" >> /home/${USERNAME}/.bashrc

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
    rm -rf /var/lib/apt/lists/* && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

RUN mkdir -p /certs && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /certs/mykey.key \
    -out /certs/mycert.pem \
    -subj "/C=GB/ST=London/L=London/O=Global Security/OU=AGI/CN=jnb.ai" && \
    chmod 755 /certs && \
    chmod 644 /certs/*

RUN python3 -m pip install --upgrade pip && \
    python3 -m pip --no-cache-dir install torch==1.13.1+cu117 torchvision==0.14.1+cu117 torchaudio==0.13.1 --extra-index-url https://download.pytorch.org/whl/cu117 && \
    python3 -m pip install --no-cache-dir \
    cmake pip pyyaml ipython notebook matplotlib black pylint graphviz pytest \
    transformers sentencepiece accelerate wandb && \
    python3 -m pip cache purge

USER $USERNAME

# Setup notebook ssl and https
RUN mkdir -p /home/${USERNAME}/.jupyter
COPY --chown=$USERNAME:$USERNAME jupyter_notebook_config.py /home/${USERNAME}/.jupyter/jupyter_notebook_config.py

ENV PATH /home/${USERNAME}/.local/bin:$PATH
WORKDIR /home/${USERNAME}

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

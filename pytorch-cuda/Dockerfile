ARG BASE_IMAGE=docker.io/pytorch/pytorch:2.1.2-cuda12.1-cudnn8-devel
FROM $BASE_IMAGE

LABEL maintainer="Dominic Kelly"

ARG DEBIAN_FRONTEND=noninteractive

ARG PYTHON_VERSION=3.11
ARG CUDA_VERSION=12.1

ARG USERNAME=dom
ARG USER_UID=1000
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV ENV_NAME="base"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
    build-essential \
    ninja-build \
    wget \
    curl \
    bzip2 \
    ca-certificates \
    git \
    htop \
    vim \
    sudo \
    tmux \
    tini \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
RUN groupadd --gid ${USER_UID} ${USERNAME} && \
    useradd -m --uid ${USER_UID} --gid ${USER_UID} -s /bin/bash -G sudo ${USERNAME} && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Ensure that non-interactive shells load /etc/profile
ENV BASH_ENV=/etc/profile

RUN mkdir -p /certs && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /certs/mykey.key \
    -out /certs/mycert.pem \
    -subj "/C=GB/ST=London/L=London/O=Global Security/OU=AGI/CN=jnb.ai" && \
    chmod 755 /certs && \
    chmod 644 /certs/*

USER $USERNAME
ENV PATH=/opt/conda/bin:$PATH

# Setup notebook ssl and https
RUN mkdir -p /home/${USERNAME}/.jupyter
COPY --chown=$USERNAME:$USERNAME jupyter_server_config.py /home/${USERNAME}/.jupyter/jupyter_server_config.py

ENV PATH=/home/${USERNAME}/.local/bin:$PATH
WORKDIR /home/${USERNAME}


RUN /opt/conda/bin/pip install --upgrade pip && \
    /opt/conda/bin/pip install --no-cache-dir \
        black cmake gcsfs ipython matplotlib notebook pylint pydantic pytest python-dotenv pyyaml requests scipy sseclient-py tqdm \
        openai \
        accelerate datasets diffusers gradio optimum peft transformers \
        bitsandbytes deepspeed einops exllamav2 sentencepiece triton wandb \
        'jupyterlab>=4.0.0,<5.0.0a0' jupyterlab-lsp 'python-lsp-server[all]' && \
    /opt/conda/bin/pip install --no-cache-dir --no-build-isolation flash-attn && \
    /opt/conda/bin/pip cache purge

# TODO add flash-attn -- need either the devel image or to build it in devel, then FROM ...-runtime and COPY
# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

ARG BASE_IMAGE=docker.io/pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime
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
    wget curl bzip2 ca-certificates \
    git \
    htop \
    vim.tiny \
    sudo \
    tmux \
    tini \
    graphviz && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
RUN groupadd --gid ${USER_UID} ${USERNAME} && \
    useradd -m --uid ${USER_UID} --gid ${USER_UID} -s /bin/bash -G sudo ${USERNAME} && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Ensure that non-interactive shells load /etc/profile
ENV BASH_ENV=/etc/profile

RUN /opt/conda/bin/pip install --upgrade pip && \
    /opt/conda/bin/pip install --no-cache-dir \
        cmake pyyaml ipython notebook matplotlib scipy black pylint graphviz pytest gcsfs\
        sentencepiece transformers accelerate diffusers wandb einops && \
    /opt/conda/bin/pip install --no-cache-dir \
        'jupyterlab>=4.0.0,<5.0.0a0' jupyterlab-lsp 'python-lsp-server[all]' && \
    /opt/conda/bin/pip cache purge

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

# Install miniai but make it happy with pytorch 2.1
RUN git clone https://github.com/fastai/course22p2 && \
    cd course22p2 && \
    sed -i -e 's/torch>=1.7,<2.1/torch>=1.7,<2.2/g' settings.ini && \
    /opt/conda/bin/pip install -e .

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

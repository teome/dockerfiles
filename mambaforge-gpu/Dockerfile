FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04
LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10
ARG CUDA_VERSION=11.8
ARG MAMBA_USER=dom

ENV CONDA_DIR=/opt/conda
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH=${CONDA_DIR}/envs/pt/bin:${CONDA_DIR}/bin:${PATH}

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
    wget bzip2 ca-certificates \
    git \
    htop \
    vim.tiny \
    tmux \
    sudo \
    tini && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --uid 1000 --shell /bin/bash ${MAMBA_USER} && \
    usermod -aG sudo ${MAMBA_USER} && \
    echo "${MAMBA_USER} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

RUN mkdir -p /certs && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /certs/mykey.key \
    -out /certs/mycert.pem \
    -subj "/C=GB/ST=London/L=London/O=Global Security/OU=AGI/CN=jnb.ai" && \
    chmod 755 /certs && \
    chmod 644 /certs/*

USER ${MAMBA_USER}

# Setup notebook ssl and https
RUN mkdir -p /home/${MAMBA_USER}/.jupyter
COPY --chown=$MAMBA_USER:$MAMBA_USER jupyter_notebook_config.py /home/${MAMBA_USER}/.jupyter/jupyter_notebook_config.py

RUN wget --no-hsts --quiet https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-$(uname -m).sh -O /tmp/miniforge.sh && \
    /bin/bash /tmp/miniforge.sh -b -p ${CONDA_DIR} && \
    rm /tmp/miniforge.sh && \
    mamba update -y mamba conda && \
    mamba install -y python=${PYTHON_VERSION} \
    pytorch torchvision torchaudio pytorch-cuda=${CUDA_VERSION} \
    cmake pyyaml ipython jupyter black pylint \
    transformers sentencepiece accelerate wandb \
    numpy mkl "libblas=*=*mkl" \
    -c pytorch-nightly -c nvidia -c conda-forge && \
    mamba clean --all --yes && \
    # find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
    # find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete && \
    # mamba clean --force-pkgs-dirs --all --yes  && \
    echo ". ${CONDA_DIR}/etc/profile.d/conda.sh && conda activate base" >> /etc/skel/.bashrc && \
    echo ". ${CONDA_DIR}/etc/profile.d/conda.sh && conda activate base" >> ~/.bashrc

WORKDIR /home/${MAMBA_USER}

ENTRYPOINT ["/opt/nvidia/nvidia_entrypoint.sh"]
CMD ["bash"]

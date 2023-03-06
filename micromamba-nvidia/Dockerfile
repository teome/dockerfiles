ARG BASE_IMAGE=docker.io/nvidia/cuda:11.8.0-runtime-ubuntu22.04

# bring in the micromamba image so we can copy files from it
FROM docker.io/mambaorg/micromamba:1.1.0 as micromamba

# This is the image we are going add micromaba to:
FROM $BASE_IMAGE

LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10
ARG CUDA_VERSION=11.8

ARG MAMBA_USER=mambauser
ARG MAMBA_USER_ID=1000
ARG MAMBA_USER_GID=1000
ENV MAMBA_USER=$MAMBA_USER
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
    wget bzip2 ca-certificates \
    git \
    htop \
    vim.tiny \
    sudo \
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

# Setup notebook ssl and https
RUN mkdir -p /root/.jupyter
COPY jupyter_notebook_config.py /root/.jupyter/jupyter_notebook_config.py
# Also for non-root
# TODO: change this to be just for one user
RUN mkdir -p /home/${MAMBA_USER}/.jupyter
COPY jupyter_notebook_config.py /home/${MAMBA_USER}/.jupyter/jupyter_notebook_config.py

COPY --from=micromamba "$MAMBA_EXE" "$MAMBA_EXE"
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_shell.sh /usr/local/bin/_dockerfile_shell.sh
COPY --from=micromamba /usr/local/bin/_entrypoint.sh /usr/local/bin/_entrypoint.sh
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_initialize_user_accounts.sh /usr/local/bin/_dockerfile_initialize_user_accounts.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_setup_root_prefix.sh /usr/local/bin/_dockerfile_setup_root_prefix.sh

RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

USER $MAMBA_USER

SHELL ["/usr/local/bin/_dockerfile_shell.sh"]

ENTRYPOINT ["/usr/local/bin/_entrypoint.sh", "/opt/nvidia/nvidia_entrypoint.sh"]
# Optional: if you want to customize the ENTRYPOINT and have a conda
# environment activated, then do this:
# ENTRYPOINT ["/usr/local/bin/_entrypoint.sh", "my_entrypoint_program"]

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

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
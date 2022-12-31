FROM nvidia/11.8.0-cudnn8-runtime-ubuntu22.04
LABEL maintainer="Dominic Kelly"

ARG PYTHON_VERSION=3.10
ARG CUDA_VERSION=3.8

ENV CONDA_DIR=/opt/conda
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH=${CONDA_DIR}/bin:${PATH}

RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
        wget bzip2 ca-certificates \
        git \
	htop \
        tini \
        > /dev/null && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    wget --no-hsts --quiet https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-$(uname -m).sh -O /tmp/miniforge.sh && \
    /bin/bash /tmp/miniforge.sh -b -p ${CONDA_DIR} && \
    rm /tmp/miniforge.sh && \
    conda install -y python=${PYTHON_VERSION} cmake conda-build pyyaml numpy mkl ipython jupyter && \
    conda install -y "libblas=*=*mkl" && \
    conda install -y pytorch torchvision torchaudio pytorch-cuda=${CUDA_VERSION} -c pytorch-nightly -c nvidia && \
    conda clean --tarballs --index-cache --packages --yes && \
    find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
    find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete && \
    conda clean --force-pkgs-dirs --all --yes  && \
    echo ". ${CONDA_DIR}/etc/profile.d/conda.sh && conda activate base" >> /etc/skel/.bashrc && \
    echo ". ${CONDA_DIR}/etc/profile.d/conda.sh && conda activate base" >> ~/.bashrc

ENTRYPOINT ["/opt/nvidia/nvidia_entrypoint.sh"]
CMD ["bash"]

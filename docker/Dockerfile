ARG CUDA_VERSION=11.3.1
ARG OS_VERSION=20.04
# pull a prebuilt image
FROM nvidia/cuda:${CUDA_VERSION}-cudnn8-devel-ubuntu${OS_VERSION}

# setup timezone
ENV TZ=US/Pacific
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ARG USER=ivanov_la
ARG UID=1063
ARG PW=mypassword
SHELL ["/bin/bash", "-c"]

# Required to build Ubuntu 20.04 without user prompts with DLFW container
ENV DEBIAN_FRONTEND=noninteractive

# Install requried libraries
RUN apt-get update && apt-get install -y software-properties-common
RUN add-apt-repository ppa:ubuntu-toolchain-r/test
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev \
    wget \
    zlib1g-dev \
    git \
    sudo \
    ssh \
    libssl-dev \
    pbzip2 \
    pv \
    bzip2 \
    unzip \
    devscripts \
    lintian \
    fakeroot \
    dh-make \
    build-essential \
    curl \
    ca-certificates \
    libx11-6 \
    nano \
    graphviz \
    libgl1-mesa-glx \
    openssh-server \
    apt-transport-https

# Install other dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgtk2.0-0 \
    libcanberra-gtk-module \
    libsm6 libxext6 libxrender-dev \
    libgtk2.0-dev pkg-config \
    libopenmpi-dev \
 && sudo rm -rf /var/lib/apt/lists/*

# Install Miniconda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.9.2-Linux-x86_64.sh \
    && mkdir /root/.conda \
    && bash Miniconda3-py38_4.9.2-Linux-x86_64.sh -b \
    && rm Miniconda3-py38_4.9.2-Linux-x86_64.sh 

ENV CONDA_DEFAULT_ENV=${project}
ENV CONDA_PREFIX=/root/miniconda3/envs/$CONDA_DEFAULT_ENV
ENV PATH=/root/miniconda3/bin:$CONDA_PREFIX/bin:$PATH

# install python 3.8
#RUN conda create -n myenv python=3.8 -y
#ENV PATH=/root/miniconda3/envs/myenv/bin:$PATH
RUN conda install python=3.8 pip

# Set environment and working directory
ENV CUDA_HOME=/usr/local/cuda
ENV LD_LIBRARY_PATH=$CUDA_HOME/lib64:$CUDA_HOME/extras/CUPTI/lib64/:$LD_LIBRARY_PATH
ENV PATH=$CUDA_HOME/bin:$PATH
ENV CFLAGS="-I$CUDA_HOME/include $CFLAGS"
ENV FORCE_CUDA="1"
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/miniconda3/envs/bin:$PATH

# install pytorch
RUN pip install torch==1.9.1+cu111 torchvision==0.10.1+cu111 torchaudio==0.9.1 -f https://download.pytorch.org/whl/torch_stable.html

# install opencv
RUN python -m pip install opencv-python==4.5.5.62

ENV PATH=/root/miniconda3/envs/bin:/root/miniconda3/bin:$PATH

# install gcc
#RUN conda install -c omgarcia gcc-6
RUN conda install -c omgarcia gcc-6 -y
# install torchpack
RUN git clone https://github.com/zhijian-liu/torchpack.git
RUN cd torchpack && python -m pip install -e .

# install mmdetection3d from source
ENV TORCH_CUDA_ARCH_LIST="7.0+PTX"
ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

# install other dependencies
RUN python -m pip install mmcv-full==1.4.0 -f https://download.openmmlab.com/mmcv/dist/cu111/torch1.9.0/index.html
RUN python -m pip install pillow==8.4.0 \
                          tqdm \
                          mmdet==2.14.0 \
                          mmsegmentation==0.14.1 \
                          numba \
                          mpi4py \
                          nuscenes-devkit \
                          setuptools==59.5.0

RUN apt-get update && apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# RUN cd mmdetection3d.git && \
#     python -m pip install --no-cache-dir -e .

# libraries path
RUN ln -s /usr/local/cuda/lib64/libcusolver.so.11 /usr/local/cuda/lib64/libcusolver.so.10

# RUN cd home/MapTR/projects/mmdet3d_plugin/maptr/modules/ops/geometric_kernel_attn && \
#     python setup.py build install

# install timm
RUN python -m pip install timm
RUN pip install yapf==0.40.1
RUN pip install wandb ipdb
RUN pip install transformers==4.38.0 einops==0.8.1

# RUN cd /home/MapTR && \
#     pip install -r requirement.txt

# additional libraries install
# RUN cd /home && git clone https://github.com/fundamentalvision/BEVFormer

# RUN pip install fvcore==0.1.5.post20221221
# RUN python -m pip install 'git+https://github.com/facebookresearch/detectron2.git'
# RUN pip install seaborn==0.12.0
USER root
RUN ["/bin/bash"]

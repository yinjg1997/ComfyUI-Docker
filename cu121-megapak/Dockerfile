################################################################################
# Dockerfile that builds 'yanwk/comfyui-boot:cu121-megapak'
# A big all-in-one package with many custom nodes.
# Using CUDA 12.1, Python 3.11, GCC 12.
# The container will be running in root (easy for rootless deploy).
################################################################################

FROM docker.io/opensuse/tumbleweed:latest

LABEL maintainer="YAN Wenkun <code@yanwk.fun>"

RUN set -eu

################################################################################
# NVIDIA CUDA devel
# Ref: https://gitlab.com/nvidia/container-images/cuda/
# Break down the steps, so we have more but smaller image layers.

RUN --mount=type=cache,target=/var/cache/zypp \
    printf "\
[cuda-opensuse15-x86_64]\n\
name=cuda-opensuse15-x86_64\n\
baseurl=https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64\n\
enabled=1\n\
gpgcheck=1\n\
gpgkey=https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/D42D0685.pub\n" \
        > /etc/zypp/repos.d/cuda-opensuse15.repo \
    && zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-cccl-12-1 \
cuda-command-line-tools-12-1 \
cuda-compat-12-1 \
cuda-cudart-12-1 \
cuda-minimal-build-12-1 \
cuda-nvcc-12-1 \
cuda-nvprof-12-1 \
cuda-nvtx-12-1 \
libcublas-12-1 \
libnpp-12-1

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-cudart-devel-12-1 \
cuda-nvml-devel-12-1 \
cuda-nvrtc-devel-12-1 \
libcublas-devel-12-1 \
libnpp-devel-12-1

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-libraries-12-1

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-libraries-devel-12-1

ENV PATH="${PATH}:/usr/local/cuda-12.1/bin" \
    LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/cuda-12.1/lib64" \
    LIBRARY_PATH="${LIBRARY_PATH}:/usr/local/cuda-12.1/lib64/stubs" \
    CUDA_HOME="/usr/local/cuda-12.1"

################################################################################
# Python and tools
# Since this image is so big, we use openSUSE-verified PIP packages for compatibility.

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper addrepo --check --refresh --priority 90 \
        'https://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Tumbleweed/Essentials/' packman-essentials \
    && zypper --gpg-auto-import-keys \
        install --no-confirm --auto-agree-with-licenses \
python311-devel \
python311-pip \
python311-wheel \
python311-setuptools \
python311-Cython \
python311-py-build-cmake \
python311-aiohttp \
python311-dbm \
python311-ffmpeg-python \
python311-GitPython \
python311-httpx \
python311-joblib \
python311-lark \
python311-matplotlib \
python311-mpmath \
python311-numba-devel \
python311-numpy1 \
python311-onnx \
python311-opencv \
python311-pandas \
python311-qrcode \
python311-rich \
python311-scikit-build \
python311-scikit-build-core-pyproject \
python311-scikit-image \
python311-scikit-learn \
python311-scipy \
python311-svglib \
python311-tqdm \
    && rm /usr/lib64/python3.11/EXTERNALLY-MANAGED \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 100

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --auto-agree-with-licenses \
Mesa-libGL1 \
Mesa-libEGL-devel \
libgthread-2_0-0 \
make \
ninja \
git \
aria2 \
fish \
fd \
vim \
opencv \
opencv-devel \
ffmpeg \
x264 \
x265 \
google-noto-sans-fonts \
google-noto-sans-cjk-fonts \
google-noto-coloremoji-fonts

################################################################################
# GCC 12 
# Required for compiling CUDA 12.1-related code.

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --auto-agree-with-licenses \
gcc12 \
gcc12-c++ \
cpp12 \
    && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-12 90 \
    && update-alternatives --install /usr/bin/cc  cc  /usr/bin/gcc-12 90 \
    && update-alternatives --install /usr/bin/cpp cpp /usr/bin/cpp-12 90 \
    && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-12 90 \
    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 90 \
    && update-alternatives --install /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-12 90 \
    && update-alternatives --install /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-12 90 \
    && update-alternatives --install /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-12 90 \
    && update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-12 90 \
    && update-alternatives --install /usr/bin/gcov-dump gcov-dump /usr/bin/gcov-dump-12 90 \
    && update-alternatives --install /usr/bin/gcov-tool gcov-tool /usr/bin/gcov-tool-12 90 

################################################################################
# Python Packages

# PyTorch, xFormers
RUN --mount=type=cache,target=/root/.cache/pip \
    pip list \
    && pip install \
        --upgrade pip wheel setuptools

ARG PIP_INDEX_URL="https://download.pytorch.org/whl/cu121"
ARG PIP_EXTRA_INDEX_URL="https://pypi.org/simple"

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        --dry-run xformers==0.0.29.post1 torch==2.5.1 torchvision torchaudio

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        --no-deps torch==2.5.1

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        --no-deps xformers==0.0.29.post1 torch==2.5.1

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        xformers==0.0.29.post1 torch==2.5.1 torchvision torchaudio

ARG PIP_INDEX_URL=
ARG PIP_EXTRA_INDEX_URL=

# Bind libs (.so files)
# Even we have CUDA installed by Zypper, we still need to install CUDA libs for Python in order to run PyTorch.
# What's more, NVIDIA's openSUSE15 repo didn't provide CuDNN & NCCL, we have to use Python packages anyway.
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}\
:/usr/local/lib64/python3.11/site-packages/torch/lib\
:/usr/local/lib/python3.11/site-packages/cusparselt/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cuda_cupti/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cuda_runtime/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cudnn/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cufft/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cublas/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cuda_nvrtc/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/curand/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cusolver/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/cusparse/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/nccl/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/nvjitlink/lib\
:/usr/local/lib/python3.11/site-packages/nvidia/nvtx/lib"


COPY builder-scripts/.  /builder-scripts/

# Deps for ComfyUI & custom nodes
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak3.txt

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak5.txt

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak7.txt

# Make sure the deps fit the needs for ComfyUI & Manager
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r https://github.com/comfyanonymous/ComfyUI/raw/refs/heads/master/requirements.txt \
        -r https://github.com/ltdrdata/ComfyUI-Manager/raw/refs/heads/main/requirements.txt \
    && pip list

################################################################################

RUN du -ah /root \
    && find /root/ -mindepth 1 -delete

COPY runner-scripts/.  /runner-scripts/

USER root
VOLUME /root
WORKDIR /root
EXPOSE 8188
ENV CLI_ARGS=""
CMD ["bash","/runner-scripts/entrypoint.sh"]

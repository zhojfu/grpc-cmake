# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.166.1/containers/cpp/.devcontainer/base.Dockerfile

# [Choice] Debian / Ubuntu version: debian-10, debian-9, ubuntu-20.04, ubuntu-18.04
ARG VARIANT="buster"
FROM mcr.microsoft.com/vscode/devcontainers/cpp:0-${VARIANT}

# [Optional] Uncomment this section to install additional packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

SHELL ["/bin/bash", "--login", "-c"]

ARG GPRC_VERSION=1.37.0
ARG NUM_JOBS=8
ARG MY_INSTALL_DIR=/usr/local

RUN apt-get update --fix-missing -y && \
    apt-get -y install \
    pkg-config \
    libopenmpi-dev \
    curl \
    wget \
    libunwind-dev \
    build-essential \
    autoconf \
    libtool \
    golang \
    dcmtk

RUN git clone --recurse-submodules -b v${GPRC_VERSION} https://github.com/grpc/grpc && \
    cd grpc && \
    mkdir -p cmake/build && \
    pushd cmake/build && \
    cmake -DgRPC_INSTALL=ON \
        -DgRPC_BUILD_TESTS=OFF \
        -DCMAKE_INSTALL_PREFIX=$MY_INSTALL_DIR \
        ../.. && \
    make -j${NUM_JOBS} && \
    make install && \
    popd && \ 
    mkdir -p third_party/abseil-cpp/cmake/build && \
    pushd third_party/abseil-cpp/cmake/build && \ 
    cmake -DCMAKE_INSTALL_PREFIX=$MY_INSTALL_DIR \
        -DCMAKE_POSITION_INDEPENDENT_CODE=TRUE \ 
        ../.. && \ 
    make -j${NUM_JOBS} && \
    make install && \
    popd
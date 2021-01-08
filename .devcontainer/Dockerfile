FROM debian:10.4-slim AS downloader
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils \
    && apt-get install -y \
    curl \
    bzip2

ARG GCC_URI=https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2
RUN mkdir -p /tmp/dc-downloads /tmp/dc-extracted/gcc /tmp/dc-extracted/cmake \
    && curl -o /tmp/dc-downloads/gcc-arm.tar.bz2  $GCC_URI \
    && bunzip2 -d /tmp/dc-downloads/gcc-arm.tar.bz2 \
    && tar -xvf /tmp/dc-downloads/gcc-arm.tar -C /tmp/dc-extracted/gcc --strip-components 1

ARG CMAKE_SCRIPT=https://cmake.org/files/v3.19/cmake-3.19.0-Linux-x86_64.sh
#ARG CMAKE_SCRIPT=https://github-production-release-asset-2e65be.s3.amazonaws.com/537699/72449280-3f8c-11eb-9e12-5bf5bf4258ea?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210107%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20210107T223214Z&X-Amz-Expires=300&X-Amz-Signature=96521d920d74366a11fd5b1a61ddc051f12fc8baec26a745d0a3055f4046f2cc&X-Amz-SignedHeaders=host&actor_id=6254153&key_id=0&repo_id=537699&response-content-disposition=attachment%3B%20filename%3Dcmake-3.19.2-Linux-x86_64.sh&response-content-type=application%2Foctet-stream
RUN curl -o /tmp/dc-downloads/cmake.sh $CMAKE_SCRIPT \
    && chmod +x /tmp/dc-downloads/cmake.sh \
    && bash /tmp/dc-downloads/cmake.sh --skip-license --prefix=/tmp/dc-extracted/cmake

FROM debian:10.4-slim AS devcontainer

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Set up non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog icu-devtools 2>&1 \
    && apt-get install -y \
    git \
    curl \
    ninja-build

RUN mkdir -p /usr/local/bin/gcc \
    && git clone https://github.com/STMicroelectronics/STM32CubeL4.git ./sources/STM32CubeL4 \
    && git clone --recursive https://github.com/azure-rtos/threadx.git ./sources/AzureRTOS

# Copy from our other container
COPY --from=downloader /tmp/dc-extracted/gcc /usr/local/bin/gcc 
COPY --from=downloader /tmp/dc-extracted/cmake/bin /usr/local/bin/cmake

# Clean up downloaded files
RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog
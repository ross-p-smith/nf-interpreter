#FROM gcc:10
FROM debian:9-slim

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Set up non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY scripts/*.sh /tmp/
RUN bash /tmp/non-root-user.sh "${USERNAME}" "${USER_UID}" "${USER_GID}" && rm /tmp/non-root-user.sh

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog icu-devtools 2>&1 \
    && apt-get install -y \
    git \
    curl \
    bzip2 \
    cmake \
    ninja-build

RUN mkdir -p /tmp/dc-downloads /usr/local/bin/gcc  \
    # Change this to programmatically get the zip rather than this hardcoded link
    # Have a look at this for reference: - https://github.com/docker-library/gcc/blob/master/10/Dockerfile
    && curl -o /tmp/dc-downloads/gcc-arm.tar.bz2 https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2 \
    && bunzip2 -d /tmp/dc-downloads/gcc-arm.tar.bz2 \
    && tar -xvf /tmp/dc-downloads/gcc-arm.tar -C /usr/local/bin/gcc --strip-components 1 \
    # Clean up downloaded files
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/dc-downloads

# Save command line history 
RUN echo "export HISTFILE=/home/$USERNAME/commandhistory/.bash_history" >> "/home/$USERNAME/.bashrc" \
     && echo "export PROMPT_COMMAND='history -a'" >> "/home/$USERNAME/.bashrc" \
     && mkdir -p /home/$USERNAME/commandhistory \
     && touch /home/$USERNAME/commandhistory/.bash_history \
     && chown -R $USERNAME /home/$USERNAME/commandhistory

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog
FROM alpine:latest

# Install basic dependencies
RUN apk update && apk add --no-cache \
    vim \
    git \
    git-lfs \
    python3 \
    py3-pip \
    curl \
    build-base \
    openssl-dev \
    pkgconfig \
    bash \
    coreutils

# Install Rust
# RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
# ENV PATH="/root/.cargo/bin:${PATH}"
# Install rust and cargo
RUN apk add --no-cache rust cargo

# Install Oh-My-bash
ENV OSH_THEME="sexy"
RUN mkdir -p /root/.oh-my-bash && bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)" --prefix=/root/.oh-my-bash
RUN cp /root/.oh-my-bash/share/oh-my-bash/bashrc ~/.bashrc

# Install nvm, node and npm
ENV NVM_DIR /root/.nvm
ENV NVM_NODEJS_ORG_MIRROR=https://unofficial-builds.nodejs.org/download/release
# ENV NODE_VERSION 20.10.0
ARG NODE_VERSION=${templateOption:nodeVersion}
ARG NVM_VERSION=${templateOption:nvmVersion}

RUN mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v$NVM_VERSION/install.sh | bash && \
    . $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    nvm use default

# Install pnpm
RUN bash -c "source $NVM_DIR/nvm.sh && npm install -g pnpm"

# Verify installations
RUN source $NVM_DIR/nvm.sh && \
    node --version && \
    npm --version && \
    rustc --version && \
    cargo --version && \
    python3 --version

# Creating a non-root user
# ********************************************************
# ARG USERNAME=user-name-goes-here
# ARG USER_UID=1000
# ARG USER_GID=$USER_UID

# # Create the user
# # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
# RUN addgroup -g $USER_GID $USERNAME && \
#     adduser -s /bin/bash -u $USER_UID -D -G $USER_GID $USERNAME && \
#     apk update && \
#     apk add --no-cache sudo && \
#     echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
#     chmod 0440 /etc/sudoers.d/$USERNAME
# # [Optional] Set the default user. Omit if you want to keep the default as root.
# # USER $USERNAME
# ********************************************************

WORKDIR /workspace

# set ENTRYPOINT for reloading nvm-environment
ENTRYPOINT ["/bin/bash", "-c", "source $NVM_DIR/nvm.sh && exec \"$@\"", "--"]

# set cmd to bash
CMD ["/bin/bash"]

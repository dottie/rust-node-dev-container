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
ARG NODE_VERSION=24
ARG NVM_VERSION=0.40.3

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

WORKDIR /workspace

# set ENTRYPOINT for reloading nvm-environment
ENTRYPOINT ["/bin/bash", "-c", "source $NVM_DIR/nvm.sh && exec \"$@\"", "--"]

# set cmd to bash
CMD ["/bin/bash"]

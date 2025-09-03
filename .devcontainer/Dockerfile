FROM ubuntu:25.04

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Arguments from devcontainer.json
ARG LOCAL_USER=ubuntu
ARG CMAKE_VERSION
ARG UV_VERSION
ARG NODE_VERSION
ARG GOLANG_VERSION
ARG RUST_VERSION

# Configure apt to avoid installing recommended packages
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-utils \
        build-essential \
        ca-certificates \
        clang \
        cmake \
        curl \
        gcc \
        g++ \
        git \
        make \
        sudo \
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js, npm, and npx if NODE_VERSION is specified
# Supports: "latest" (current), "lts" (latest LTS), or specific version like "v20.14.0"
# (npm and npx are automatically included with Node.js installation)
RUN if [ -n "$NODE_VERSION" ]; then \
        if [ "$NODE_VERSION" = "latest" ]; then \
            # Install the latest Node.js version
            curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash - && \
            sudo apt-get install -y nodejs; \
        elif [ "$NODE_VERSION" = "lts" ]; then \
            # Install the latest LTS Node.js version
            curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - && \
            sudo apt-get install -y nodejs; \
        else \
            # For specific versions, extract major version number (e.g., v20.14.0 -> 20)
            NODE_MAJOR_VERSION=$(echo $NODE_VERSION | sed -e 's/^v//' -e 's/\.[0-9]*\.[0-9]*$//'); \
            curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR_VERSION}.x | sudo -E bash - && \
            sudo apt-get install -y nodejs; \
        fi \
    fi

# Install CMake if version is specified
RUN if [ -n "$CMAKE_VERSION" ]; then \
        if [ "$CMAKE_VERSION" = "latest" ]; then \
            # Get latest version tag from GitHub releases
            CMAKE_LATEST_VERSION=$(curl -s https://api.github.com/repos/Kitware/CMake/releases | grep -E '"tag_name"' | head -1 | cut -d '"' -f 4); \
            CMAKE_VERSION_NO_V=$(echo $CMAKE_LATEST_VERSION | sed 's/^v//'); \
            wget -O cmake.sh https://github.com/Kitware/CMake/releases/download/${CMAKE_LATEST_VERSION}/cmake-${CMAKE_VERSION_NO_V}-linux-x86_64.sh && \
            chmod +x cmake.sh && \
            sudo ./cmake.sh --skip-license --prefix=/usr/local && \
            rm cmake.sh; \
        else \
            CMAKE_VERSION_NO_V=$(echo $CMAKE_VERSION | sed 's/^v//'); \
            wget -O cmake.sh https://github.com/Kitware/CMake/releases/download/${CMAKE_VERSION}/cmake-${CMAKE_VERSION_NO_V}-linux-x86_64.sh && \
            chmod +x cmake.sh && \
            sudo ./cmake.sh --skip-license --prefix=/usr/local && \
            rm cmake.sh; \
        fi \
    fi

# Install uv if UV_VERSION is specified
# Supports: "latest" or specific version like "0.8.15"
RUN if [ -n "$UV_VERSION" ]; then \
        if [ "$UV_VERSION" = "latest" ]; then \
            # Get latest version tag from GitHub releases
            UV_LATEST_VERSION=$(curl -s https://api.github.com/repos/astral-sh/uv/releases/latest | grep -E '"tag_name"' | cut -d '"' -f 4); \
            wget -O uv.tar.gz https://github.com/astral-sh/uv/releases/download/${UV_LATEST_VERSION}/uv-x86_64-unknown-linux-gnu.tar.gz && \
            tar -xzf uv.tar.gz && \
            sudo cp uv-x86_64-unknown-linux-gnu/uv /usr/bin/ && \
            sudo cp uv-x86_64-unknown-linux-gnu/uvx /usr/bin/ && \
            rm -rf uv-x86_64-unknown-linux-gnu && \
            rm uv.tar.gz; \
        else \
            wget -O uv.tar.gz https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-x86_64-unknown-linux-gnu.tar.gz && \
            tar -xzf uv.tar.gz && \
            sudo cp uv-x86_64-unknown-linux-gnu/uv /usr/bin/ && \
            sudo cp uv-x86_64-unknown-linux-gnu/uvx /usr/bin/ && \
            rm -rf uv-x86_64-unknown-linux-gnu && \
            rm uv.tar.gz; \
        fi \
    fi

# Install Go if GOLANG_VERSION is specified
# Supports: "latest" or specific version like "go1.25.0" or "go1.24.6"
RUN if [ -n "$GOLANG_VERSION" ]; then \
        if [ "$GOLANG_VERSION" = "latest" ]; then \
            # Get latest Go version from the download page
            GO_LATEST_VERSION=$(curl -s https://go.dev/dl/ | grep -E -o 'go[0-9]+\.[0-9]+\.[0-9]+\.linux-amd64\.tar\.gz' | head -1); \
            wget -O go.tar.gz https://go.dev/dl/${GO_LATEST_VERSION} && \
            sudo tar -C /usr/local -xzf go.tar.gz && \
            rm go.tar.gz; \
        else \
            wget -O go.tar.gz https://go.dev/dl/${GOLANG_VERSION}.linux-amd64.tar.gz && \
            sudo tar -C /usr/local -xzf go.tar.gz && \
            rm go.tar.gz; \
        fi \
    fi

# Copy bash theme snippet to /tmp and then append it to /etc/environment
COPY bash_theme_snippet.sh /tmp/
RUN cat /tmp/bash_theme_snippet.sh >> /etc/environment

# Create user based on LOCAL_USER argument
ARG LOCAL_USER
ARG USERNAME=${LOCAL_USER:-ubuntu}

RUN sudo deluser --remove-home ubuntu 2>/dev/null || true \
    && sudo groupadd --gid 1000 $USERNAME 2>/dev/null || true \
    && sudo useradd --uid 1000 --gid 1000 -m -s "/bin/bash" $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL | sudo tee /etc/sudoers.d/$USERNAME \
    && sudo chmod 0440 /etc/sudoers.d/$USERNAME

# Set default user
USER $USERNAME

# Install Rust if RUST_VERSION is specified
# Supports: "latest" (stable toolchain) or specific version like "1.89.0"
RUN if [ -n "$RUST_VERSION" ]; then \
        # Install rustup for the current user
        curl -fsSL https://sh.rustup.rs | sh -s -- -y --no-modify-path --default-toolchain none; \
        # Source the environment to make rustup available
        . "$HOME/.cargo/env"; \
        # Install the specified Rust version
        if [ "$RUST_VERSION" = "latest" ]; then \
            rustup install stable; \
            rustup default stable; \
        else \
            rustup install $RUST_VERSION; \
            rustup default $RUST_VERSION; \
        fi; \
        # Copy Rust binaries to system-wide location
        sudo cp -r ~/.cargo/bin/* /usr/local/bin/; \
    fi

# Copy bash theme snippet to /tmp and then append it to /etc/environment
COPY bash_theme_snippet.sh /tmp/
RUN cat /tmp/bash_theme_snippet.sh >> /home/$USERNAME/.bashrc

# Add Go to PATH environment variable if it was installed
ENV PATH="/usr/local/go/bin:${PATH}"

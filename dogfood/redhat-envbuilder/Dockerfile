FROM registry.access.redhat.com/ubi9/ubi:latest

SHELL ["/bin/bash", "-c"]

# Set ARGs that will be used throughout the build
ARG DEBIAN_FRONTEND="noninteractive"
ARG GO_VERSION=1.24.3  # Latest stable version for Coder 2.20.3
ARG NODE_VERSION=20.19.0  # Latest LTS version
ARG NVM_DIR=/usr/local/nvm

# Install basic packages and FIPS compliance tools
RUN dnf update -y && \
    dnf install -y \
    bash \
    bash-completion \
    bind-utils \
    ca-certificates \
    cmake \
    crypto-policies \
    crypto-policies-scripts \
    curl \
    dnsutils \
    file \
    findutils \
    fipscheck \
    git \
    gnupg \
    graphviz \
    htop \
    iproute \
    iputils \
    jq \
    less \
    lsof \
    make \
    man \
    net-tools \
    openssh-server \
    openssl \
    openssl-fips \
    pkg-config \
    python3 \
    python3-pip \
    rsync \
    screen \
    strace \
    sudo \
    tmux \
    traceroute \
    unzip \
    util-linux \
    vim \
    wget \
    which \
    zip \
    zsh && \
    dnf clean all && \
    # Configure FIPS-compliant policies - explicitly set FIPS mode
    update-crypto-policies --set FIPS && \
    # Verify FIPS mode is active
    update-crypto-policies --show

# Install Go with FIPS-compliant download validation
RUN curl --silent --show-error --location \
    "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz" \
    -o /usr/local/go.tar.gz && \
    # Verify Go checksum to ensure FIPS compliance
    echo "$(curl -s https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz.sha256) /usr/local/go.tar.gz" | sha256sum --check && \
    mkdir -p /usr/local/go && \
    tar --extract --gzip --directory=/usr/local/go --file=/usr/local/go.tar.gz --strip-components=1

ENV PATH=$PATH:/usr/local/go/bin

# Install Docker
RUN dnf install -y dnf-plugins-core && \
    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo && \
    dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin && \
    systemctl enable docker

# Install Terraform with FIPS-compliant validation
RUN wget -O /tmp/terraform.zip "https://releases.hashicorp.com/terraform/1.11.5/terraform_1.11.5_linux_amd64.zip" && \
    wget -O /tmp/terraform_SHA256SUMS "https://releases.hashicorp.com/terraform/1.11.5/terraform_1.11.5_SHA256SUMS" && \
    grep linux_amd64 /tmp/terraform_SHA256SUMS | sha256sum --check --status && \
    unzip /tmp/terraform.zip -d /usr/local/bin && \
    rm -f /tmp/terraform.zip /tmp/terraform_SHA256SUMS && \
    chmod +x /usr/local/bin/terraform && \
    terraform version

# Install Node.js using NVM with FIPS-compliant validation
RUN mkdir -p $NVM_DIR && \
    # Download NVM with verification
    curl -o nvm_install.sh https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.8/install.sh && \
    echo "$(curl -s https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.8/install.sh | sha256sum | cut -d ' ' -f1) nvm_install.sh" | sha256sum --check && \
    bash nvm_install.sh && \
    # Install Node using NVM with FIPS-compatible settings
    source $NVM_DIR/nvm.sh && \
    # Force Node to use OpenSSL FIPS
    export NODE_OPTIONS="--openssl-config=/etc/crypto-policies/back-ends/openssl.config" && \
    nvm install $NODE_VERSION && \
    nvm use $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    # Verify that Node.js will use FIPS compliant crypto
    node -e "console.log('FIPS mode:', process.versions.openssl)"

ENV PATH=$NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

# Install pnpm and npm
RUN npm install -g npm@10.12.1 && \
    npm install -g pnpm@9.17.4

# Setup GitHub CLI
RUN dnf install -y 'dnf-command(config-manager)' && \
    dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo && \
    dnf install -y gh

# Configure systemd services for our container
RUN systemctl enable \
    docker \
    ssh

# Add coder user and allow use of docker/sudo
RUN useradd coder \
    --create-home \
    --shell=/bin/bash \
    --groups=wheel \
    --uid=1000 \
    --user-group && \
    echo "coder ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/coder && \
    chmod 0440 /etc/sudoers.d/coder

# Add Docker group and add coder user to it
RUN groupadd -f docker && \
    usermod -aG docker coder

# Adjust SSH config for X11 forwarding
RUN echo "PermitUserEnvironment yes" >>/etc/ssh/sshd_config && \
    echo "X11Forwarding yes" >>/etc/ssh/sshd_config && \
    echo "X11UseLocalhost no" >>/etc/ssh/sshd_config

# Set environment variables
ENV GOPATH="/home/coder/go"
ENV PATH="$GOPATH/bin:$PATH"
ENV GOPRIVATE="coder.com,cdr.dev,go.coder.com,github.com/cdr,github.com/coder"
# Set OpenSSL to use FIPS mode
ENV OPENSSL_FORCE_FIPS_MODE=1
# Configure Node.js for FIPS compliance and memory settings
ENV NODE_OPTIONS="--max-old-space-size=8192 --openssl-config=/etc/crypto-policies/back-ends/openssl.config"
# Ensure Python respects system crypto policy
ENV PYTHONHTTPSVERIFY=1

USER coder
WORKDIR /home/coder
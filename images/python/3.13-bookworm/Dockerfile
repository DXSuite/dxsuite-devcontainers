# Use the official Python 3.13.5 image on Debian Bookworm.
FROM python:3.13.5-bookworm


####################################################################################################
# METADATA
####################################################################################################
# Set metadata for the Docker image.
LABEL org.opencontainers.image.title="dxsuite-devcontainer-python" \
      org.opencontainers.image.version="0.0.1" \
      org.opencontainers.image.description="DXSuite all-in-one Python 3.13 devcontainer on Debian Bookworm." \
      org.opencontainers.image.authors="DXSuite" \
      org.opencontainers.image.source="https://github.com/DXSuite/dxsuite-devcontainers" \
      org.opencontainers.image.base.name="python" \
      org.opencontainers.image.base.version="3.13.5-bookworm"


####################################################################################################
# ENVIRONMENT CONFIGURATION
####################################################################################################
# Configure the container's environment. We'll set up a non-root user,
# switch to Zsh for a better shell, and set the timezone.
USER root

# Increase system-wide limits (these act as a fallback if Docker limits aren't set)
# Note: Docker compose ulimits take precedence over these settings, we do this however
# as there's older versions of Docker, Podman etc. that don't support the ulimits setting.
RUN echo "* soft nofile 65535" >> /etc/security/limits.conf && \
    echo "* hard nofile 65535" >> /etc/security/limits.conf && \
    echo "* soft nproc 32768" >> /etc/security/limits.conf && \
    echo "* hard nproc 32768" >> /etc/security/limits.conf && \
    echo "* soft memlock unlimited" >> /etc/security/limits.conf && \
    echo "* hard memlock unlimited" >> /etc/security/limits.conf && \
    echo "fs.file-max = 65535" >> /etc/sysctl.conf && \
    echo "kernel.pid_max = 4194304" >> /etc/sysctl.conf

# Set the non-root user name
ENV NON_ROOT_USER="vscode"

# Set environment variable to help identify when running in the container
ENV DEVCONTAINER=true

## Set timezone non-interactively
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

## Create timezone file and set localtime
RUN echo $TZ >/etc/timezone && \
    ln -sf /usr/share/zoneinfo/$TZ /etc/localtime


####################################################################################################
# SYSTEM DEPENDENCIES
####################################################################################################
# Install essential packages for building software, development, and various command-line tools.
# They are grouped by function.

## Install all dependencies
RUN apt update && \
    apt upgrade -y && \
    apt install -y --fix-missing \
    `# Build tools` \
    autoconf \
    automake \
    build-essential \
    ca-certificates \
    cmake \
    git \
    gpg \
    libtool \
    meson \
    ninja-build \
    pkg-config \
    ruby \
    texinfo \
    \
    `# Development libraries` \
    gnutls-dev \
    libfreetype6-dev \
    libgnutls28-dev \
    libsdl2-dev \
    libunistring-dev \
    libxcb-shm0-dev \
    libxcb-xfixes0-dev \
    libxcb1-dev \
    zlib1g-dev \
    \
    `# Python` \
    pipx \
    python3-pip \
    \
    `# Command-line tools` \
    bat \
    curl \
    gh \
    gnupg2 \
    gnutls-bin \
    htop \
    jq \
    less \
    man-db \
    nano \
    procps \
    ripgrep \
    shellcheck \
    shfmt \
    sqlite3 \
    sshpass \
    sudo \
    tmux \
    unzip \
    vim \
    wget \
    xvfb \
    xxd \
    yasm \
    \
    `# Networking tools` \
    aggregate \
    dnsutils \
    iproute2 \
    ipset \
    iptables \
    locales \
    \
    `# Shell and environment` \
    tree \
    zsh \
    zsh-syntax-highlighting \
    \
    `# Fonts` \
    fonts-liberation \
    fonts-liberation2 \
    fonts-powerline \
    && apt autoremove -y \
    && apt autoclean \
    && rm -rf /var/lib/apt/lists/*


## Set Locale
RUN echo "en_US.UTF-8 UTF-8" | tee -a /etc/locale.gen
RUN locale-gen

## Update fonts cache
RUN fc-cache -f -v

## Update CA Certs
RUN update-ca-certificates

####################################################################################################
# EZA INSTALLATION
####################################################################################################
# Install eza - a modern replacement for 'ls' with better defaults and additional features.
# This requires adding the eza community repository and its GPG key for package verification.
RUN sudo mkdir -p /etc/apt/keyrings \
    && wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | sudo gpg --dearmor -o /etc/apt/keyrings/gierens.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/gierens.gpg] http://deb.gierens.de stable main" | sudo tee /etc/apt/sources.list.d/gierens.list \
    && sudo chmod 644 /etc/apt/keyrings/gierens.gpg /etc/apt/sources.list.d/gierens.list \
    && sudo apt update \
    && sudo apt install -y eza

####################################################################################################
# FZF INSTALLATION
####################################################################################################
# Install fzf - a command-line fuzzy finder for searching files, command history, and more.
# Downloads and installs the appropriate binary for the current architecture from GitHub releases.
ARG FZF_VERSION="0.64.0"
RUN ARCH=$(dpkg --print-architecture) && \
    case "$ARCH" in \
    amd64) FZF_ARCH="amd64" ;; \
    arm64) FZF_ARCH="arm64" ;; \
    armhf) FZF_ARCH="armv7" ;; \
    armel) FZF_ARCH="armv6" ;; \
    ppc64el) FZF_ARCH="ppc64le" ;; \
    s390x) FZF_ARCH="s390x" ;; \
    *) echo "Unsupported architecture: $ARCH" && exit 1 ;; \
    esac && \
    wget "https://github.com/junegunn/fzf/releases/download/v${FZF_VERSION}/fzf-${FZF_VERSION}-linux_${FZF_ARCH}.tar.gz" && \
    tar -xzf "fzf-${FZF_VERSION}-linux_${FZF_ARCH}.tar.gz" -C /usr/bin/ && \
    rm "fzf-${FZF_VERSION}-linux_${FZF_ARCH}.tar.gz"

####################################################################################################
# GIT DELTA INSTALLATION
####################################################################################################
# Install Git Delta for better-looking git diffs in the terminal.
# Downloads and installs the appropriate .deb package for the current architecture from GitHub releases.
ARG DELTA_VERSION="0.18.2"
RUN ARCH=$(dpkg --print-architecture) && \
    case "$ARCH" in \
    amd64) DELTA_ARCH="amd64" ;; \
    arm64) DELTA_ARCH="arm64" ;; \
    armhf) DELTA_ARCH="armhf" ;; \
    i386) DELTA_ARCH="i386" ;; \
    *) echo "Unsupported architecture: $ARCH" && exit 1 ;; \
    esac && \
    wget "https://github.com/dandavison/delta/releases/download/${DELTA_VERSION}/git-delta_${DELTA_VERSION}_${DELTA_ARCH}.deb" && \
    sudo dpkg -i "git-delta_${DELTA_VERSION}_${DELTA_ARCH}.deb" && \
    rm "git-delta_${DELTA_VERSION}_${DELTA_ARCH}.deb"

####################################################################################################
# NON-ROOT USER SETUP
####################################################################################################
# Create a non-root user to avoid running as root (a security best practice).
# This also helps with file permissions when mounting local files.
# The user gets sudo access for when it's needed.
#
# Create the user with the same UID as the host user.
RUN useradd -g users -m "$NON_ROOT_USER"

# Set the password for the non-root user
RUN echo "$NON_ROOT_USER:$NON_ROOT_USER" | chpasswd

# Add user to sudo group
RUN usermod -a -G sudo "$NON_ROOT_USER"

# Add to sudoers
RUN echo "$NON_ROOT_USER" ALL=\(root\) NOPASSWD:ALL >"/etc/sudoers.d/$NON_ROOT_USER"

# Create command history directory for VS Code devcontainer compatibility
RUN mkdir -p /commandhistory && \
    touch /commandhistory/.zsh_history && \
    touch /commandhistory/.bash_history && \
    chown -R $NON_ROOT_USER:users /commandhistory

####################################################################################################
# ZSH INITIALISATION
####################################################################################################
# Configure Zsh for both root and the non-root user
# using the recommended zshrc as a starting point.
#
# ZSH configuration for root user
RUN cp /etc/zsh/newuser.zshrc.recommended /root/.zshrc

# Configure root user's shell history (both ZSH and Bash)
RUN echo 'export HISTFILE="/commandhistory/.zsh_history"' >> /root/.zshrc && \
    echo 'HISTSIZE=100000' >> /root/.zshrc && \
    echo 'SAVEHIST=100000' >> /root/.zshrc && \
    echo 'setopt HIST_IGNORE_DUPS' >> /root/.zshrc && \
    echo 'setopt SHARE_HISTORY' >> /root/.zshrc && \
    echo 'setopt APPEND_HISTORY' >> /root/.zshrc && \
    echo 'setopt INC_APPEND_HISTORY' >> /root/.zshrc && \
    echo 'export PROMPT_COMMAND="history -a"' >> /root/.bashrc && \
    echo 'export HISTFILE="/commandhistory/.bash_history"' >> /root/.bashrc

# Switch to the non-root user for subsequent operations.
USER $NON_ROOT_USER

# ZSH configuration for non-root user
RUN cp /etc/zsh/newuser.zshrc.recommended "$HOME/.zshrc"

# Switch to ZSH as the default shell.
SHELL ["/bin/zsh", "-l", "-c"]

# Set the default shell to zsh for the non-root user
ENV SHELL="/bin/zsh"

# Set the home directory for the non-root user and change the working directory.
ENV HOME="/home/$NON_ROOT_USER"
WORKDIR $HOME

####################################################################################################
# ZSH CONFIGURATION
####################################################################################################
# Install Oh My Zsh
RUN sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# Configure zshrc history settings
RUN tee -a "$HOME/.zshrc" <<-'EOF' >/dev/null
# History settings - VS Code devcontainer compatible
export HISTFILE="/commandhistory/.zsh_history"
HISTSIZE=100000
SAVEHIST=100000
setopt HIST_IGNORE_DUPS
setopt SHARE_HISTORY
setopt APPEND_HISTORY
setopt INC_APPEND_HISTORY

EOF

# Configure bash history settings (for compatibility)
RUN echo 'export PROMPT_COMMAND="history -a"' >> "$HOME/.bashrc" && \
    echo 'export HISTFILE="/commandhistory/.bash_history"' >> "$HOME/.bashrc"


# Copy the P10K theme to the home directory
COPY ./assets/.p10k.zsh "$HOME/.p10k.zsh"

# Install P10K and replace the default theme
RUN git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k" && \
    sed -i 's/ZSH_THEME="robbyrussell"/#ZSH_THEME="robbyrussell"/' "$HOME/.zshrc"

# Add P10K config header & footer to zshrc
RUN CURRENT_ZSHRC=$(cat "$HOME/.zshrc") && \
    cat > "$HOME/.zshrc" <<-EOF

## POWERLEVEL10K
# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
# Initialization code that may require console input (password prompts, [y/n]
# confirmations, etc.) must go above this block; everything else may go below.
if [[ ! \$COMPOSER_NO_INTERACTION == "1" ]]; then
    if [[ -r "\${XDG_CACHE_HOME:-\$HOME/.cache}/p10k-instant-prompt-\${(%):-%n}.zsh" ]]; then
        source "\${XDG_CACHE_HOME:-\$HOME/.cache}/p10k-instant-prompt-\${(%):-%n}.zsh"
    fi
fi

## FZF
source <(fzf --zsh)

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time Oh My Zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes

if [[ ! \$COMPOSER_NO_INTERACTION == "1" ]]; then
    # P10K Theme
    ZSH_THEME="powerlevel10k/powerlevel10k"
else
    # Simplified Theme
    ZSH_THEME="robbyrussell"
fi

$CURRENT_ZSHRC

## INITIALISE P10K
if [[ ! \$COMPOSER_NO_INTERACTION == "1" ]]; then
    # To customize prompt, run \`p10k configure\` or edit ~/.p10k.zsh.
    [[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
fi

## Aliases
alias p10k-down='prompt_powerlevel9k_teardown'
alias p10k-up='prompt_powerlevel9k_setup'

## EZA - LS Replacement
# OMZ ZSH Adds the following defaults, we will overide these as we want to use eza instead:
#   alias lsa='ls -lah'
#   alias l='ls -lah'
#   alias ll='ls -lh'
#   alias la='ls -lAh'
#
# Common Args:
#   eza --long --all --tree --icons --total-size -level=1 --show-symlinks --only-dirs --only-files --sort Name

# Use eza via the `l` alias - bundle sensible defaults
alias l="eza --sort Name --group-directories-first --classify auto --color auto --icons auto --octal-permissions --git --time-style long-iso --header"

# Replaces: ls -a / ls -l / ls -al 
alias ll='l --long'
alias la='l --long --almost-all'
alias lsa='l --long --all'
alias lsal='l --long --all'

# Gives total file size of subdirectories (can be slow)
alias lts="l --long --all --total-size"

# Replaces tree / tree --only dirs
alias lt="l --long --all --tree --level=5"
alias ltd="l --long --all --tree --show-symlinks --only-dirs --level=5"

EOF

# Install OMZ Plugins and configure plugins
RUN git clone https://github.com/zsh-users/zsh-autosuggestions "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions" && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting" && \
    sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting ssh)/' "$HOME/.zshrc"

####################################################################################################
# NODE.JS ENVIRONMENT (NVM)
####################################################################################################
# Install NVM to manage Node.js versions. We'll install a specific Node.js
# version and make it the default.
ARG NVM_VERSION="0.40.3"
ARG NODE_VERSION="24.3.0"

# Set NVM directory
ENV NVM_DIR="$HOME/.nvm"

# Install NVM
RUN mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v${NVM_VERSION}/install.sh | bash

# Install Node.js and update NPM to latest version
RUN source $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    nvm use default && \
    nvm install-latest-npm

# Set Node.js memory limit to 8GB
ENV NODE_OPTIONS="--max-old-space-size=8192"

# Ensure Node binaries are on PATH even for non-interactive commands
ENV NVM_SYMLINK_CURRENT=true
ENV NVM_BIN="$NVM_DIR/versions/node/current/bin" \
    PATH="$NVM_BIN:$PATH"

####################################################################################################
# UV INSTALLATION
####################################################################################################
# Install UV for Python package management.
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

####################################################################################################
# PYTHON TOOLING (PIPX)
####################################################################################################
# Configure Pipx to install and run Python tools in isolated environments.
# This avoids messing with system-wide Python packages.
# The PATH is already configured above for oh-my-posh which shares the same ~/.local/bin directory.
RUN pipx ensurepath

####################################################################################################
# FINALISE CONTAINER
####################################################################################################
# Override the Python base image's ENTRYPOINT to ensure we get a shell
ENTRYPOINT []
# Start a Zsh shell by default when the container runs.
CMD ["/bin/zsh", "-l"]

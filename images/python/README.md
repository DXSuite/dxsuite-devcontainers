# DXSuite Python Development Container

[![Python](https://img.shields.io/badge/Python-3.13.5-blue.svg)](https://www.python.org/)
[![Debian](https://img.shields.io/badge/Debian-Bookworm-red.svg)](https://www.debian.org/)
[![Docker](https://img.shields.io/badge/Docker-Container-blue.svg)](https://www.docker.com/)

A comprehensive, pre-configured Python development container built on Debian Bookworm, designed for modern development workflows with an exceptional terminal experience powered by **[ZSH](https://www.zsh.org/)**, **[Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)**, and **[Powerlevel10k](https://github.com/romkatv/powerlevel10k)**.

Part of the Developer Experience Suite (DXSuite) - professionally crafted development containers.

## 📑 Table of Contents

- [🌟 Key Features](#-key-features)
- [🚀 Quick Start](#-quick-start)
- [🏗️ Using as a Base Image](#️-using-as-a-base-image)
- [🎨 Terminal Experience](#-terminal-experience)
- [🐚 Shell Features & Tools](#-shell-features)
- [🐍 Python Development](#-python-development)
- [🟢 Node.js Development](#-nodejs-development)
- [📦 Version & Tagging Strategy](#-version--tagging-strategy)
- [⚙️ Advanced Configuration](#️-advanced-configuration)
- [💡 Tips & Tricks](#-tips--tricks)
- [🤝 Contributing](#-contributing)

## 🌟 Key Features

- **🚀 Modern Shell Experience**: Pre-configured ZSH with Oh My Zsh framework and Powerlevel10k theme for a beautiful, informative prompt
- **⚡ Lightning-Fast Python**: Includes [UV](https://github.com/astral-sh/uv) package manager (10-100x faster than pip)
- **🔍 Powerful Search**: [ripgrep](https://github.com/BurntSushi/ripgrep) and [fzf](https://github.com/junegunn/fzf) for blazing-fast file and history search
- **📁 Enhanced File Browsing**: [eza](https://github.com/eza-community/eza) with Git integration and file icons
- **🎨 Beautiful Git Diffs**: [delta](https://github.com/dandavison/delta) for syntax-highlighted, side-by-side diffs
- **🔧 Developer Tools**: Pre-installed build tools, Python 3.13.5, Node.js 24.3.0 (via NVM), and 50+ development utilities
- **🔒 Non-Root User**: Secure setup with `vscode` user and passwordless sudo

## 🚀 Quick Start

### What You Get Out of the Box

When you start this container, you immediately have:

1. **Beautiful Terminal**: Powerlevel10k prompt showing Git status, Python environment, execution times
2. **Smart Command Completion**: Type partial commands and press `Tab` for intelligent suggestions
3. **Visual Feedback**: Commands turn green when valid, red when invalid (before you even run them!)
4. **Instant File Search**: Press `Ctrl+T` to fuzzy search any file in your project
5. **Git Superpowers**: See file changes with `l` (enhanced ls), view diffs with syntax highlighting

### Getting Started

```bash
# Pull the latest version
docker pull ghcr.io/dxsuite/dxsuite-devcontainer-python:latest

# Run interactively with command history persistence
docker run -it --rm \
  -v $(pwd):/workspace \
  -v ${PWD##*/}-commandhistory:/commandhistory \
  ghcr.io/dxsuite/dxsuite-devcontainer-python:latest

# Use with VS Code Dev Containers
# Create .devcontainer/devcontainer.json pointing to this image
```

<details>
<summary><b>Building Locally</b></summary>

If you want to build the image locally with proper names:

```bash
# Using Docker directly (from repository root)
docker build -t dxsuite-devcontainer-python:latest ./images/python/3.13-bookworm/

# With multiple tags
docker build -t dxsuite-devcontainer-python:latest \
             -t dxsuite-devcontainer-python:3.13-bookworm \
             -t dxsuite-devcontainer-python:3.13.5-bookworm \
             ./images/python/3.13-bookworm/
```

**Running your locally built image with command history:**

```bash
# Run with automatic volume naming (based on current directory)
docker run -it --rm \
  -v $(pwd):/workspace \
  -v ${PWD##*/}-commandhistory:/commandhistory \
  dxsuite-devcontainer-python:latest

# Or with a specific volume name
docker run -it --rm \
  -v $(pwd):/workspace \
  -v myproject-commandhistory:/commandhistory \
  dxsuite-devcontainer-python:latest
```

💡 **Tip**: Without the `-v` mount for `/commandhistory`, your shell history won't persist between container runs!

</details>

## 🏗️ Using as a Base Image

Build your own custom development container on top of DXSuite's foundation:

### 1. Create Your Dockerfile

```dockerfile
# Use DXSuite Python container as base
FROM ghcr.io/dxsuite/dxsuite-devcontainer-python:latest

# Switch to root for installations
USER root

# Install your project-specific dependencies
RUN apt-get update && apt-get install -y \
    postgresql-client \
    redis-tools \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages globally or for the user
USER vscode
RUN pip install --user django djangorestframework celery

# Copy your project files (if building with project context)
WORKDIR /workspace
COPY --chown=vscode:users . .

# Install project dependencies
RUN pip install --user -r requirements.txt
```

### 2. Build Your Custom Image

```bash
# Build your custom container
docker build -t myproject-dev .

# Run with your customizations
docker run -it --rm -v $(pwd):/workspace myproject-dev
```

### 3. VS Code Dev Container Configuration

Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "My Project Dev Container",
  "build": {
    "dockerfile": "../Dockerfile",
    "context": ".."
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance"
      ]
    }
  },
  "forwardPorts": [8000],
  "postCreateCommand": "pip install --user -e .",
  "remoteUser": "vscode",
  // Mount a volume for command history persistence
  "mounts": [
    "source=${localWorkspaceFolderBasename}-commandhistory,target=/commandhistory,type=volume"
  ]
}
```


### 4. Using with Docker Compose

If you prefer Docker Compose, create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  dev:
    build: .
    image: myproject-dev
    volumes:
      # Mount your project files
      - .:/workspace:cached
      # IMPORTANT: Mount volume for command history persistence
      - commandhistory:/commandhistory
    working_dir: /workspace
    command: /bin/zsh
    stdin_open: true
    tty: true
    user: vscode

volumes:
  # Define the named volume for command history
  commandhistory:
```

Run with: `docker-compose run --rm dev`

#### What You Inherit

When using this as a base image, you automatically get:

✅ **Shell Environment**: ZSH, Oh My Zsh, Powerlevel10k, all plugins configured
✅ **Development Tools**: All 50+ pre-installed tools (ripgrep, fzf, delta, etc.)
✅ **Python Setup**: Python 3.13.5, UV, pipx, pip ready to use
✅ **Node.js**: NVM with Node.js 24.3.0 pre-installed
✅ **User Configuration**: `vscode` user with sudo access
✅ **History Persistence**: `/commandhistory` configured (requires volume mount - see examples above)
✅ **Git Enhancements**: Delta diffs and 150+ git aliases

#### ⚠️ Important: Command History Persistence

The container is pre-configured to store command history in `/commandhistory`, but **you must mount a volume** to persist it across container rebuilds:

**Without a volume mount**: History is lost when the container is removed
**With a volume mount**: History persists forever, even across rebuilds

The examples above include the necessary volume configuration:
- **VS Code**: The `mounts` array in `.devcontainer/devcontainer.json`
- **Docker Compose**: The `volumes` section with `commandhistory:/commandhistory`
- **Docker CLI**: Add `-v myproject-history:/commandhistory`

This follows the [VS Code standard](https://code.visualstudio.com/remote/advancedcontainers/persist-bash-history) for persisting shell history.

#### Best Practices

1. **Always switch users appropriately**:
   ```dockerfile
   USER root  # For system packages
   # ... install system dependencies ...
   USER vscode  # For user packages
   ```

2. **Preserve the working directory**:
   ```dockerfile
   WORKDIR /workspace  # Standard mount point
   ```

3. **Use `--user` flag for pip installs** when running as vscode:
   ```dockerfile
   RUN pip install --user package-name
   ```

4. **Layer your changes efficiently**:
   - System packages first (less likely to change)
   - Language packages next
   - Application code last (most likely to change)


## 🎨 Terminal Experience

### Why This Container?

This container transforms your development experience with a meticulously configured terminal environment:

- **[ZSH](https://www.zsh.org/)**: Advanced shell with superior auto-completion, globbing patterns, and scripting capabilities
- **[Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)**: The most popular ZSH framework with 300+ plugins and 140+ themes
- **[Powerlevel10k](https://github.com/romkatv/powerlevel10k)**: Lightning-fast theme with:
  - Git status indicators in your prompt
  - Python virtual environment detection
  - Command execution time tracking
  - Current directory with intelligent truncation
  - Pre-configured with sensible defaults (no setup wizard needed!)

### Additional Terminal Enhancements

- **Auto-suggestions**: Fish-like command suggestions as you type (via [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions))
- **Syntax Highlighting**: Real-time command validation with color coding (via [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting))
- **Smart History**: 100,000 lines of searchable, deduplicated command history
- **Git Aliases**: 100+ pre-configured Git shortcuts (e.g., `gst` for `git status`)

<details>
<summary><b>Configuration & Font Setup</b></summary>

### Commands

| Command | Description |
|---------|-------------|
| `p10k configure` | Customize prompt appearance |
| `p10k-down` | Temporarily disable Powerlevel10k theme |
| `p10k-up` | Re-enable Powerlevel10k theme |

### Required Fonts for Powerlevel10k

**Important**: Powerlevel10k requires special fonts to display icons correctly. Without these fonts, you'll see broken characters.

1. **Download the required fonts:**
   - [MesloLGS NF Regular](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
   - [MesloLGS NF Bold](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
   - [MesloLGS NF Italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
   - [MesloLGS NF Bold Italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

2. **Install on your system:**
   - **macOS:** Double-click `.ttf` files → "Install Font"
   - **Linux:** Copy to `~/.local/share/fonts/` and run `fc-cache -f -v`
   - **Windows:** Right-click `.ttf` files → "Install"

### IDE Configuration

<details>
<summary><b>VS Code</b></summary>

Add to your `settings.json`:
```json
{
  "terminal.integrated.fontFamily": "'MesloLGS NF', Menlo, Monaco, 'Courier New', monospace",
  "terminal.integrated.fontSize": 14
}
```

Or via UI: Settings → Terminal → Font Family → Enter `'MesloLGS NF'`
</details>

<details>
<summary><b>JetBrains IDEs</b></summary>

1. Go to Settings/Preferences → Editor → Font → Console Font
2. Select "MesloLGS NF" from the dropdown
3. Adjust size as needed (recommended: 14)
</details>

<details>
<summary><b>Terminal.app (macOS)</b></summary>

1. Terminal → Preferences → Profiles → Text
2. Click "Change..." under Font
3. Select "MesloLGS NF"
</details>

</details>

## 🐚 Shell Features

The container uses **ZSH** as the default shell, enhanced with [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) and [Powerlevel10k](https://github.com/romkatv/powerlevel10k) for a powerful, modern terminal experience.

### Shell Enhancements

| Feature | Description | Official Repo |
|---------|-------------|---------------|
| **ZSH** | Advanced shell with better completion, globbing, and scripting | [zsh.org](https://www.zsh.org/) |
| **Oh My Zsh** | Framework for managing ZSH configuration | [GitHub](https://github.com/ohmyzsh/ohmyzsh) |
| **Powerlevel10k** | Fast, flexible, and beautiful ZSH theme | [GitHub](https://github.com/romkatv/powerlevel10k) |

### Pre-Configured ZSH Plugins

The following Oh My Zsh plugins are pre-installed and configured:

| Plugin | Description | Features | Official Repo |
|--------|-------------|----------|---------------|
| **git** | Git aliases and functions | `gst` (status), `gco` (checkout), `gp` (push), `gd` (diff), `gcmsg` (commit -m) | [Built-in](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git) |
| **zsh-autosuggestions** | Fish-like command suggestions | Type any command and see grayed-out suggestions from your history. Press `→` to accept | [GitHub](https://github.com/zsh-users/zsh-autosuggestions) |
| **zsh-syntax-highlighting** | Real-time syntax validation | Commands turn green when valid, red when invalid. Strings, options, and paths are color-coded | [GitHub](https://github.com/zsh-users/zsh-syntax-highlighting) |
| **ssh** | SSH agent management | Automatically starts ssh-agent and loads your SSH keys on shell startup | [Built-in](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/ssh) |

### Command Line Tools

#### Fuzzy Finding with FZF

[FZF](https://github.com/junegunn/fzf) is pre-installed and integrated with ZSH for powerful fuzzy finding:

| Keybinding | Function | Description |
|------------|----------|-------------|
| `Ctrl+T` | File finder | Fuzzy find files and directories |
| `Ctrl+R` | History search | Search command history interactively |
| `Alt+C` | Directory navigation | Change to any directory quickly |

#### Enhanced Diffs with Delta

[Git Delta](https://github.com/dandavison/delta) transforms your Git experience:
- **Syntax Highlighting**: Full language-aware highlighting in diffs (300+ languages)
- **Side-by-Side Mode**: View changes in parallel with `git diff --side-by-side`
- **Line Numbers**: Clear before/after line numbers in every diff
- **File Headers**: Beautifully styled file paths and metadata
- **Theme Support**: Adapts to your terminal's color scheme
- **Improved Navigation**: Better keyboard navigation through large diffs

### Shell Configuration

#### Powerlevel10k Benefits

The included `.p10k.zsh` configuration provides:
- **Instant Prompt**: Terminal ready in milliseconds, not seconds
- **Transient Prompt**: Previous commands condensed to save screen space
- **Smart Git Integration**: Shows branch, dirty state, commits ahead/behind
- **Python Environment**: Automatically displays active virtualenv/conda
- **Error Indicators**: Exit codes shown for failed commands
- **Time Tracking**: Execution time for long-running commands

#### History Management
- **Capacity**: 100,000 lines saved to disk
- **Persistence**: History stored in `/commandhistory` (VS Code devcontainer standard)
- **Volume Mount**: Compatible with VS Code's automatic history volume mounting
- **Dual Shell Support**: Works with both ZSH (`.zsh_history`) and Bash (`.bash_history`)
- **Sharing**: History shared across all terminal sessions
- **Smart Deduplication**: Consecutive duplicates automatically removed
- **Search**: `Ctrl+R` for fuzzy history search with fzf

**Note**: When using this as a base image with VS Code Dev Containers, history will automatically persist across container rebuilds using VS Code's standard volume mounting.

#### Shell Features
- **Locale**: UTF-8 enabled (`en_US.UTF-8`) for proper character support
- **Default User**: `vscode` with passwordless sudo
- **Working Directory**: Remembers last location between sessions

## 🛠️ Included Tools & Aliases

### Enhanced File Listing with eza

The container includes `eza` (a modern alternative to `ls`) with convenient aliases that provide enhanced file listings with Git integration, icons, and tree views.

| Alias | Description | Replaces |
|-------|-------------|----------|
| `l` | Basic listing with sensible defaults | `ls` |
| `ll` | Long format with details | `ls -l` |
| `la` | Long format, almost all files | `ls -la` |
| `lsa` | Long format, all files | `ls -lsa` |
| `lsal` | Long format, all files (alternative) | `ls -lsa` |
| `lts` | Show total size of subdirectories | - |
| `lt` | Tree view (5 levels deep) | `tree` |
| `ltd` | Tree view, directories only | `tree -d` |

**eza default options:**
- 📁 **Icons**: Visual file type indicators
- 🔀 **Git Integration**: See modified (M), new (?), ignored files inline
- 🔐 **Permissions**: Octal format (e.g., `755`) for clarity
- 📅 **Timestamps**: ISO format for consistency
- 📂 **Smart Sorting**: Directories first, then files alphabetically
- 🎨 **Colors**: Different colors for different file types

**Example output:**
```
Permissions Size User  Date Modified Name
.rwxr-xr-x   147 user  2024-01-15    📁 src/
.rw-r--r--  1.2k user  2024-01-15    📄 README.md
.rw-r--r--   456 user  2024-01-14    🐍 main.py [M]
```

<details>
<summary><b>Complete Tool List (50+ tools)</b></summary>

#### Build Tools
- `autoconf`, `automake`, `cmake`, `meson`, `ninja-build`
- `gcc`, `g++`, `make` (via build-essential)
- `pkg-config`, `libtool`
- `yasm`: Assembler for x86 architectures

#### Language Environments
- **Python 3.13.5**: With `pip`, `pipx`, `uv`
- **Node.js 24.3.0**: Via NVM with npm
- **Ruby**: With gem support

#### CLI Productivity
- **[ripgrep](https://github.com/BurntSushi/ripgrep)** (`rg`): Ultra-fast recursive search
- **[fzf](https://github.com/junegunn/fzf)**: Command-line fuzzy finder
- **[bat](https://github.com/sharkdp/bat)**: A `cat` clone with syntax highlighting and Git integration
- **[delta](https://github.com/dandavison/delta)**: Syntax-highlighting pager for git diffs
- **[eza](https://github.com/eza-community/eza)**: Modern `ls` replacement with colors and Git status
- **htop**: Interactive process viewer with real-time metrics
- **tmux**: Terminal multiplexer for managing multiple sessions
- **[jq](https://github.com/jqlang/jq)**: Command-line JSON processor
- **yq**: YAML processor (similar to jq but for YAML)

#### Text Processing & Documentation
- **less**: Pager for viewing long text files
- **man-db**: Manual page reader and database
- **texinfo**: GNU documentation system
- **xxd**: Hex dump utility for binary file analysis

#### Development Utilities
- **git**: Version control with delta integration
- **[gh](https://github.com/cli/cli)**: GitHub's official CLI
- **[shellcheck](https://github.com/koalaman/shellcheck)**: Static analysis for shell scripts
- **[shfmt](https://github.com/mvdan/sh)**: Shell script formatter
- **vim**: Vi IMproved text editor
- **nano**: User-friendly text editor
- **sqlite3**: Command-line interface for SQLite databases

#### System Utilities
- **procps**: Process monitoring tools (`ps`, `top`, `kill`, etc.)
- **tree**: Directory structure visualization
- **unzip**: Archive extraction utility
- **sshpass**: Non-interactive SSH password authentication
- **xvfb**: X Virtual Framebuffer for headless GUI testing

#### Network & System
- `curl`, `wget`: HTTP clients for downloading files
- `dnsutils`: DNS utilities (`dig`, `nslookup`, `host`)
- `iproute2`: Modern networking tools (`ip`, `ss`)
- `aggregate`: CIDR network aggregation tool
- `ipset`: IP set administration tool
- `iptables`: Packet filtering framework
- `sudo`: Execute commands with elevated privileges
- `ca-certificates`: Common CA certificates
- `gnupg2`, `gpg`: GNU Privacy Guard for encryption
- `gnutls-bin`: GnuTLS command-line tools

#### Font Support
- `fonts-liberation`, `fonts-liberation2`: Liberation font families
- `fonts-powerline`: Powerline fonts for terminal themes
</details>

## 🐍 Python Development

The container includes Python 3.13.5 with modern tooling and package managers:

### Package Management Tools

| Tool | Purpose | Usage |
|------|---------|-------|
| **[UV](https://github.com/astral-sh/uv)** | Ultra-fast Python package installer (10-100x faster than pip) | `uv pip install package` |
| **[pipx](https://github.com/pypa/pipx)** | Install Python CLI tools in isolated environments | `pipx install tool` |
| **pip** | Traditional Python package manager | `pip install package` |
| **python3-pip** | Python 3 pip package (system-wide) | Pre-installed |

### Example Usage

```bash
# UV - Lightning fast package installation
uv pip install requests numpy pandas

# pipx - Install CLI tools without conflicts
pipx install poetry
pipx install black
pipx install ruff
pipx install mypy

# Traditional pip
pip install --user package-name
```

### Python Configuration
- **Version**: Python 3.13.5 (latest stable)
- **Base Image**: Official Python on Debian Bookworm
- **pipx Path**: Automatically configured in PATH
- **Global pipx**: Available for system-wide tools

## 🟢 Node.js Development

The container includes Node.js managed through NVM (Node Version Manager):

### Node.js Configuration

| Component | Version | Details |
|-----------|---------|---------|
| **[NVM](https://github.com/nvm-sh/nvm)** | 0.40.3 | Node version management |
| **Node.js** | 24.3.0 | Latest LTS version |
| **npm** | Latest | Updated to latest version |

### Features
- **Memory Limit**: 8GB heap size configured (`NODE_OPTIONS=--max-old-space-size=8192`)
- **Path Configuration**: Node binaries automatically in PATH
- **NVM Integration**: Full NVM commands available in shell

### NVM Commands

```bash
# List installed versions
nvm list

# Install a different version
nvm install 20.9.0

# Switch versions
nvm use 20.9.0

# Set default version
nvm alias default 24.3.0
```

## 📦 Version & Tagging Strategy

We maintain the following tags for flexibility and stability:

| Tag | Example | When to Use |
|-----|---------|-------------|
| **Latest** | `latest` | Always get the newest version |
| **Python Version** | `3.13-bookworm` | Track Python minor version |
| **Specific Version** | `3.13.5-bookworm` | Pin to exact Python version |

```bash
# Examples
docker pull ghcr.io/dxsuite/dxsuite-devcontainer-python:latest
docker pull ghcr.io/dxsuite/dxsuite-devcontainer-python:3.13-bookworm
docker pull ghcr.io/dxsuite/dxsuite-devcontainer-python:3.13.5-bookworm
```

## ⚙️ Advanced Configuration

<details>
<summary><b>System Configuration</b></summary>

### Resource Limits

The container is configured with increased system limits for development workloads:

| Resource | Soft Limit | Hard Limit | Description |
|----------|------------|------------|-------------|
| **Open Files** | 65,535 | 65,535 | Maximum file descriptors |
| **Processes** | 32,768 | 32,768 | Maximum user processes |
| **Memory Lock** | Unlimited | Unlimited | Memory that can be locked |

Additional kernel parameters:
- `fs.file-max`: 65,535 (system-wide file descriptor limit)
- `kernel.pid_max`: 4,194,304 (maximum PID value)

### Environment Variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `DEVCONTAINER` | `true` | Identifies when running in the container |
| `TZ` | `UTC` | Default timezone setting |
| `DEBIAN_FRONTEND` | `noninteractive` | Prevents interactive prompts during builds |
| `SHELL` | `/bin/zsh` | Default shell for the container |
| `NODE_OPTIONS` | `--max-old-space-size=8192` | Node.js memory limit (8GB) |
| `NVM_SYMLINK_CURRENT` | `true` | Ensures Node binaries are accessible |

### User Configuration

- **Default User**: `vscode` (non-root)
- **User ID**: Automatically assigned
- **Groups**: `users`, `sudo`
- **Sudo Access**: Passwordless sudo configured
- **Home Directory**: `/home/vscode`
- **Default Password**: Same as username (for sudo when needed)

### Locale Configuration

- **Default Locale**: `en_US.UTF-8`
- **Character Encoding**: UTF-8
- **Timezone**: UTC (configurable via `TZ` environment variable)

</details>

<details>
<summary><b>Development Libraries</b></summary>

The container includes essential development libraries for compiling and building software:

### Graphics & Media Libraries
- **libfreetype6-dev**: Font rendering library
- **libsdl2-dev**: Simple DirectMedia Layer for multimedia applications
- **libxcb-shm0-dev**, **libxcb-xfixes0-dev**, **libxcb1-dev**: X11 XCB libraries for GUI applications

### Security & Encryption
- **gnutls-dev**, **libgnutls28-dev**: GNU TLS library for secure communications
- **gnutls-bin**: Command-line tools for TLS/SSL

### System Libraries
- **libunistring-dev**: Unicode string library
- **zlib1g-dev**: Compression library

These libraries enable compilation of a wide range of software including GUI applications, secure network tools, and multimedia software.

</details>

<details>
<summary><b>Git Configuration</b></summary>

### Git Delta Integration

The container comes with [Git Delta](https://github.com/dandavison/delta) pre-configured as the default pager for Git, providing:

- **Syntax Highlighting**: Language-aware highlighting for 300+ file types
- **Line Numbers**: Clear line numbering in diffs
- **Side-by-Side View**: Compare changes side-by-side with `--side-by-side`
- **File Headers**: Styled file paths and commit information
- **Improved Navigation**: Better keyboard navigation through diffs

### Using Git Delta

```bash
# Delta is automatically used for these commands:
git diff              # See unstaged changes
git show             # View commit details
git log -p           # Show commit history with patches
git blame            # Enhanced blame output

# Side-by-side view
git diff --side-by-side

# Customize delta on the fly
git -c delta.line-numbers=false diff
```

### Git Productivity Boost

The pre-configured `git` plugin provides 150+ aliases. Here are the most useful ones:

| Alias | Command | Description |
|-------|---------|-------------|
| `gst` | `git status` | Show working tree status |
| `ga` | `git add` | Stage changes |
| `gc` | `git commit` | Commit changes |
| `gp` | `git push` | Push to remote |
| `gl` | `git pull` | Pull from remote |
| `gco` | `git checkout` | Switch branches |
| `gb` | `git branch` | List/manage branches |
| `gd` | `git diff` | Show changes |
| `gaa` | `git add --all` | Stage all changes |
| `gcmsg` | `git commit -m` | Commit with message |
| `glog` | `git log --graph` | Visual commit history |
| `grhh` | `git reset --hard HEAD` | Reset to HEAD |
| `gclean` | `git clean -fd` | Remove untracked files |

Run `alias | grep git` to see all 150+ Git aliases.

</details>

<details>
<summary><b>SSH Configuration</b></summary>

### SSH Agent Management

The pre-configured `ssh` plugin from Oh My Zsh provides automatic SSH key management:

**What it does:**
- **Auto-starts ssh-agent**: No need to manually start the SSH agent
- **Persists agent**: Reuses the same agent across terminal sessions
- **Auto-loads keys**: Automatically adds your SSH keys from `~/.ssh`
- **Secure**: Agent forwarding ready for remote development

**Key Features:**

1. **Zero Configuration**: Works out of the box - just place your keys in `~/.ssh/`
2. **Smart Key Loading**: Only loads keys when needed, not on every shell start
3. **Agent Persistence**: One agent for all your terminals (no duplicate agents)
4. **Passphrase Caching**: Enter your passphrase once per session

**Common SSH Commands Enhanced:**
```bash
# Check loaded keys
ssh-add -l

# Add a specific key
ssh-add ~/.ssh/id_ed25519

# Forward agent when connecting (-A flag)
ssh -A user@remote-host

# Test GitHub connection
ssh -T git@github.com
```

**Best Practices:**
- Store keys in standard locations: `~/.ssh/id_rsa`, `~/.ssh/id_ed25519`
- Use `ssh-keygen -t ed25519` for new keys (more secure than RSA)
- Set appropriate permissions: `chmod 600 ~/.ssh/id_*`

</details>

<details>
<summary><b>Project Structure</b></summary>

```
3.13-bookworm/
├── Dockerfile              # Container definition
├── VERSION                 # Semantic version tracking
└── assets/
    └── .p10k.zsh          # Powerlevel10k configuration
```

</details>

## 💡 Tips & Tricks

### Maximize Your Productivity

1. **Learn the Keybindings**:
   - `Ctrl+T`: Fuzzy find files
   - `Ctrl+R`: Search command history
   - `Alt+C`: Navigate to any directory
   - `→` (right arrow): Accept autosuggestion

2. **Use the Aliases**:
   - `l` instead of `ls` for beautiful file listings
   - `gst` instead of `git status`
   - `rg` for ultra-fast code searching

3. **Customize Your Experience**:
   - Run `p10k configure` to personalize your prompt
   - Edit `~/.zshrc` to add your own aliases
   - Install additional Oh My Zsh plugins

### Performance Notes

- First shell startup may take a few seconds (building font cache)
- Subsequent shells start instantly thanks to Powerlevel10k's instant prompt
- The container is optimized for both interactive and non-interactive use

## 🤝 Contributing

This project is maintained by DXSuite. To report issues or contribute:

- Repository: [github.com/DXSuite/dxsuite-devcontainers](https://github.com/DXSuite/dxsuite-devcontainers)
- Issues: [GitHub Issues](https://github.com/DXSuite/dxsuite-devcontainers/issues)

## 📜 License

This project is provided as-is for development use. See the repository for license details.

---

Built with ❤️ for developers everywhere.
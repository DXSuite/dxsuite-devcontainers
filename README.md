# DXSuite Development Containers

[![Docker](https://img.shields.io/badge/Docker-Containers-blue.svg)](https://www.docker.com/)
[![GitHub](https://img.shields.io/badge/GitHub-Container%20Registry-black.svg)](https://ghcr.io)

A collection of professionally crafted development containers for modern development workflows, part of the Developer Experience Suite (DXSuite).

## 🏗️ Repository Structure

This monorepo contains multiple development container images organized by language and runtime:

```
dxsuite-devcontainers/
├── images/
│   ├── python/                    # Python development containers
│   │   ├── 3.13-bookworm/        # Python 3.13.5 on Debian Bookworm
│   │   │   ├── Dockerfile
│   │   │   ├── VERSION
│   │   │   └── assets/
│   │   └── README.md
│   ├── node/                     # Node.js containers (future)
│   └── java/                     # Java containers (future)
├── .github/
│   └── workflows/
│       └── publish-python.yml    # Python image CI/CD
├── LICENSE
└── README.md                     # This file
```

## 📦 Available Images

### Python Development Containers

| Image | Base | Python Version | Features | Registry |
|-------|------|----------------|----------|----------|
| **Python 3.13 Bookworm** | Debian Bookworm | 3.13.5 | ZSH, UV, Node.js, 50+ tools | `ghcr.io/dxsuite/dxsuite-devcontainer-python` |

📚 **[Full Python Container Documentation →](images/python/README.md)** - Comprehensive guide with 700+ lines of detailed information about tools, configuration, and usage.

More languages and versions coming soon!

## 🚀 Quick Start

### Using Pre-built Images

```bash
# Pull the latest Python container
docker pull ghcr.io/dxsuite/dxsuite-devcontainer-python:latest

# Run interactively
docker run -it --rm \
  -v $(pwd):/workspace \
  -v ${PWD##*/}-commandhistory:/commandhistory \
  ghcr.io/dxsuite/dxsuite-devcontainer-python:latest
```

### Building Locally

```bash
# Clone the repository
git clone https://github.com/DXSuite/dxsuite-devcontainers.git
cd dxsuite-devcontainers

# Build a specific image (e.g., Python 3.13)
docker build -t dxsuite-devcontainer-python:latest ./images/python/3.13-bookworm/

# Run your locally built image
docker run -it --rm -v $(pwd):/workspace dxsuite-devcontainer-python:latest
```

## 🛠️ Development Workflow

### VS Code Dev Containers

Create a `.devcontainer/devcontainer.json` in your project:

```json
{
  "name": "Python Development",
  "image": "ghcr.io/dxsuite/dxsuite-devcontainer-python:latest",
  "features": {},
  "mounts": [
    "source=${localWorkspaceFolderBasename}-commandhistory,target=/commandhistory,type=volume"
  ]
}
```

### Using as Base Images

Create a custom Dockerfile for your project:

```dockerfile
FROM ghcr.io/dxsuite/dxsuite-devcontainer-python:latest

# Switch to root for system packages
USER root

# Install project-specific dependencies
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Switch back to vscode user
USER vscode

# Install Python packages
RUN pip install --user django djangorestframework
```

## 🏭 Adding New Images

To add a new language or version:

1. **Create the directory structure**:
   ```bash
   mkdir -p images/[language]/[version]/
   ```

2. **Add required files**:
   - `Dockerfile` - Container definition
   - `VERSION` - Version identifier
   - `assets/` - Configuration files (if needed)

3. **Create a workflow** (if new language):
   ```bash
   cp .github/workflows/publish-python.yml .github/workflows/publish-[language].yml
   # Edit the workflow for your new language
   ```

4. **Update documentation**:
   - Add README.md in `images/[language]/`
   - Update this main README.md

## 📋 Image Specifications

All DXSuite containers follow these standards:

- **Base OS**: Debian-based (preferably Bookworm)
- **Shell**: ZSH with Oh My Zsh and Powerlevel10k
- **User**: Non-root `vscode` user with sudo access
- **History**: Persistent command history via `/commandhistory`
- **Tools**: Language-specific toolchain plus common development utilities
- **Multi-platform**: Support for both AMD64 and ARM64

## 🔄 CI/CD Pipeline

Each image type has its own GitHub Actions workflow that:

1. Triggers on changes to relevant files
2. Builds multi-platform images (AMD64 & ARM64)
3. Publishes to GitHub Container Registry (ghcr.io)
4. Tags with appropriate version identifiers

Workflows use a matrix strategy to build multiple versions efficiently.

## 📚 Documentation

- **Language-specific docs**: See README.md in each `images/[language]/` directory
- **Container usage**: Detailed in each language's documentation
- **Customization**: Examples provided for extending base images
- **Best practices**: Included in language-specific guides

## 🤝 Contributing

We welcome contributions! To add new images or improve existing ones:

1. Fork the repository
2. Create a feature branch
3. Add your changes following the structure above
4. Submit a pull request

Please ensure:
- Dockerfiles are well-commented
- Documentation is updated
- Images follow DXSuite standards
- Multi-platform builds work correctly

## 📜 License

See [LICENSE](LICENSE) file for details.

## 🔗 Links

- **Repository**: [github.com/DXSuite/dxsuite-devcontainers](https://github.com/DXSuite/dxsuite-devcontainers)
- **Registry**: [ghcr.io/dxsuite](https://ghcr.io/dxsuite)
- **Issues**: [GitHub Issues](https://github.com/DXSuite/dxsuite-devcontainers/issues)

---

Built with ❤️ for developers everywhere.
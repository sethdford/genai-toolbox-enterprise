# Enterprise Installation Guide

Quick installation guide for enterprise users who need to get started without Go build tools.

## 🚀 Recommended: One-Line Install

**Fastest way to get started** - Automatically downloads the correct binary for your platform:

```bash
curl -fsSL https://raw.githubusercontent.com/sethdford/genai-toolbox/main/scripts/install.sh | bash
```

**What happens:**
1. Detects your OS (macOS, Linux, Windows) and architecture (Intel/ARM)
2. Downloads the latest pre-built binary from GitHub Releases
3. Installs to `~/.local/bin/genai-toolbox`
4. Makes the binary executable

**Custom install location:**
```bash
# Install to /usr/local/bin (requires sudo)
INSTALL_DIR=/usr/local/bin curl -fsSL https://raw.githubusercontent.com/sethdford/genai-toolbox/main/scripts/install.sh | bash

# Install to custom directory
INSTALL_DIR=$HOME/bin curl -fsSL https://raw.githubusercontent.com/sethdford/genai-toolbox/main/scripts/install.sh | bash
```

**Add to PATH** (if installed to ~/.local/bin):
```bash
# Add to ~/.bashrc or ~/.zshrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 📦 NPM Install (Node.js Users)

If your team uses Node.js/npm, this is the easiest option:

```bash
# Global installation
npm install -g @genai-toolbox/server

# Verify installation
genai-toolbox --version
```

**Use without installing:**
```bash
# Run directly with npx
npx @genai-toolbox/server --tools-file tools.yaml
```

**Benefits:**
- Familiar npm workflow
- Automatic updates with `npm update -g`
- Works with package.json scripts
- No manual binary management

---

## ⬇️ Direct Binary Download

For manual installation or air-gapped environments:

### Step 1: Download

Visit [Releases](https://github.com/sethdford/genai-toolbox/releases/latest) and download the archive for your platform:

- **macOS (Apple Silicon)**: `genai-toolbox-darwin-arm64.tar.gz`
- **macOS (Intel)**: `genai-toolbox-darwin-amd64.tar.gz`
- **Linux (amd64)**: `genai-toolbox-linux-amd64.tar.gz`
- **Linux (arm64)**: `genai-toolbox-linux-arm64.tar.gz`
- **Windows (amd64)**: `genai-toolbox-windows-amd64.zip`

### Step 2: Extract

**macOS/Linux:**
```bash
tar -xzf genai-toolbox-*.tar.gz
chmod +x genai-toolbox
sudo mv genai-toolbox /usr/local/bin/
```

**Windows:**
```powershell
# Extract the zip file
# Move genai-toolbox.exe to a directory in your PATH
# Example: C:\Program Files\genai-toolbox\
```

### Step 3: Verify

```bash
genai-toolbox --version
```

---

## 🐳 Docker/Container

Perfect for containerized deployments:

```bash
docker pull us-central1-docker.pkg.dev/database-toolbox/toolbox/toolbox:0.21.0
```

Run with your tools.yaml:
```bash
docker run -p 5000:5000 \
  -v $(pwd)/tools.yaml:/app/tools.yaml \
  us-central1-docker.pkg.dev/database-toolbox/toolbox/toolbox:0.21.0 \
  --tools-file /app/tools.yaml
```

---

## 🍺 Homebrew (macOS/Linux)

If you use Homebrew:

```bash
brew install mcp-toolbox
```

---

## ✅ Verify Installation

After installation, verify everything works:

```bash
# Check version
genai-toolbox --version

# View help
genai-toolbox --help

# Test with example config
genai-toolbox --tools-file examples/tools.yaml
```

---

## 🚀 Quick Start

Once installed, create a `tools.yaml` configuration:

```yaml
sources:
  - name: my-dynamodb
    kind: dynamodb
    region: us-east-1

tools:
  list-tables:
    kind: dynamodb-list-tables
    source: my-dynamodb
    description: List all DynamoDB tables

toolsets:
  aws:
    - list-tables
```

Start the server:

```bash
genai-toolbox --tools-file tools.yaml
```

The server starts on `http://localhost:5000`

---

## 📚 Next Steps

- **Configuration**: See [Configuration Guide](README.md#configuration)
- **AWS Setup**: See [AWS Integration Guide](docs/guides/AWS_INTEGRATIONS.md)
- **Production**: See [Production Deployment Guide](docs/guides/PRODUCTION_DEPLOYMENT_GUIDE.md)
- **Validation**: See [Validation Guide](docs/guides/VALIDATION_GUIDE.md)

---

## 🆘 Troubleshooting

### "Command not found" after install

Add the install directory to your PATH:

```bash
# If installed to ~/.local/bin
export PATH="$HOME/.local/bin:$PATH"

# Make permanent
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Permission denied

Make the binary executable:

```bash
chmod +x /path/to/genai-toolbox
```

### NPM install fails

Try with sudo (not recommended) or use npx:

```bash
npx @genai-toolbox/server --tools-file tools.yaml
```

### Behind corporate proxy

Set proxy environment variables before installing:

```bash
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
curl -fsSL https://raw.githubusercontent.com/sethdford/genai-toolbox/main/scripts/install.sh | bash
```

---

## 🔐 Enterprise Considerations

### Air-Gapped Environments

1. Download binary on internet-connected machine
2. Verify checksum from release page
3. Transfer to air-gapped environment
4. Extract and install manually

### Security Scanning

All binaries are built via GitHub Actions:
- Source: Public repository
- Build: Reproducible GitHub Actions
- Checksums: Available in release assets

### Version Pinning

For production, pin to specific versions:

```bash
# Install specific version
VERSION=0.21.0 curl -fsSL https://raw.githubusercontent.com/sethdford/genai-toolbox/main/scripts/install.sh | bash

# NPM specific version
npm install -g @genai-toolbox/server@0.21.0
```

---

## 📞 Support

- **Issues**: https://github.com/sethdford/genai-toolbox/issues
- **Documentation**: https://googleapis.github.io/genai-toolbox/
- **Discord**: https://discord.gg/GQrFB3Ec3W

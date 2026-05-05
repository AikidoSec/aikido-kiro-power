# Aikido Security - Kiro Powers

Brings [Aikido Security](https://aikido.dev) directly into [Kiro](https://kiro.dev) with two powers: one scans code for SAST vulnerabilities, exposed secrets, and IaC misconfigurations and helps fix them, while the other lists and triages Aikido issues for a repository or selected files.

## What it does

- `power-aikido-security-scan`
    - **SAST** — Static analysis to catch common vulnerability patterns
    - **Secrets detection** — Finds exposed API keys, tokens, and credentials
    - **IaC scanning** — Detects misconfigurations in infrastructure-as-code files
- `power-aikido-list-issues` - List, count, and summarize Aikido issues for a repository or selected files.

## Prerequisites

- Node.js 18+
- An [Aikido Security](https://app.aikido.dev) account

## Setup

### 1. Install the Power

See [Kiro Docs](https://kiro.dev/docs/powers/installation/) for detailed instructions on how to setup Kiro Powers.

### 2. Get your API key

Go to [app.aikido.dev](https://app.aikido.dev) → **Settings** → **Integrations** → **IDE Plugins** and copy your API key.

### 3. Configure the API key

**Option A — Environment variable (recommended)**

```bash
export AIKIDO_API_KEY=your_api_key_here
```

Add this to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.) to make it permanent.

**Option B — Hardcode in MCP config**

Open `~/.kiro/settings/mcp.json` and set the key directly in the `power` section.

## Usage

Once installed, Kiro can be prompted to scan or list files for security issues and resolve them. When vulnerabilities are found, Kiro will explain each issue and apply fixes guided by Aikido's remediation advice. A hook will be added as well that triggers Kiro to run Aikido scans after it has made changes to or added code.

### Security scan power

Use prompts like:

> "Scan file.js for security vulnerabilities"
> "Scan my changed files for security vulnerabilities"

### Issue listing power

Use prompts like:

> "List Aikido issues in this repository"
> "Show high severity issues in src/auth.js"


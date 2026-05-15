# Aikido Security - Kiro Powers

Brings [Aikido Security](https://aikido.dev) directly into [Kiro](https://kiro.dev) with two powers: one scans code for SAST vulnerabilities, exposed secrets, and IaC misconfigurations and helps fix them; the other lists issues from your Aikido feed.

Both powers use the [Aikido MCP Server](https://www.npmjs.com/package/@aikidosec/mcp) (`@aikidosec/mcp`), launched via `npx`.

## Powers

| Power | Folder | What it does |
|-------|--------|--------------|
| **Scan code with Aikido Security** | `power-aikido-security-scan` | Runs local SAST, secrets, and IaC scans on code you write or modify, explains findings, and guides fixes |
| **List Aikido Issues** | `power-aikido-list-issues` | Fetches issues from the Aikido feed for listing, counting and summarizing |

### Scan power (`power-aikido-security-scan`)

- **SAST** — Static analysis for common vulnerability patterns
- **Secrets detection** — Finds exposed API keys, tokens, and credentials
- **IaC scanning** — Detects misconfigurations in infrastructure-as-code files

**MCP tool:** `aikido_full_scan` — scans up to 50 files per request (batch larger sets into multiple calls).

### List issues power (`power-aikido-list-issues`)

Fetches security issues from the **Aikido feed**, with optional scope by repository, cloud, VM, domain, or container.

**MCP tool:** `aikido_issues_list` — supports filters such as `repo_name`, `cloud_name`, `vm_name`, `domain_name`, `container_name`, and `issue_types` (e.g. `sast`, `leaked_secret`, `iac`, `cloud`, `open_source` and more).

## Prerequisites

- Node.js 18+
- An [Aikido Security](https://app.aikido.dev) account

## Setup

### 1. Install the powers

See [Kiro Docs — Powers installation](https://kiro.dev/docs/powers/installation/) for how to install powers from this repository.

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

Once installed, prompt Kiro to scan code or list feed issues. When vulnerabilities are found during a scan, Kiro explains each issue and can apply fixes using Aikido’s remediation guidance, then rescan to verify.

### Security scan

- “Scan `file.js` for security vulnerabilities”
- “Scan my changed files for security vulnerabilities”

### Issue listing

> "Scan file.js for security vulnerabilities"
> "Scan my changed files for security vulnerabilities"

### Issue listing power

Use prompts like:

> “List Aikido issues for domain 'my-example.com'"
> “List all Aikido security issues with type 'malware' for this repo"


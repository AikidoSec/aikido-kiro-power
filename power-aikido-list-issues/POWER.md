---
name: "aikido-list-issues"
displayName: "List Aikido Issues"
description: "Fetches security issues from the Aikido feed so you can list, count, summarize, triage, or plan fixes—with optional scope by cloud, repository, VM, domain, or container."
keywords: ["aikido", "issues", "feed", "security", "triage", "vulnerabilities", "secrets", "list", "count", "summarize", "cloud", "repository", "vm", "domain", "container"]
author: Aikido Security
---

# Purpose

Use this power when the user asks to list, show, count, or summarize **Aikido security issues from the Aikido feed**, or when you need the current issue set before remediation.

# Onboarding

Before proceeding, validate that the user has completed the following steps:

## Step 1

Prompt the user to configure their own Aikido API key for authentication. They can get an Aikido API key at [app.aikido.dev](https://app.aikido.dev/settings/integrations/ide/kiro) -> Settings -> Integrations -> IDE Plugins.
Once they have their API key they can set it as an environment variable named AIKIDO_API_KEY on their system, or hardcode it directly into the user level MCP configuration file (usually at ~/.kiro/settings/mcp.json) in the power section. Guide the user on how to do this.

## Step 2

Validate user has Node.js 18+, required to run the Aikido MCP server.

## Step 3

Verify the MCP server is reachable and the API key is valid by calling **aikido_full_scan** with a minimal test payload: one file with path `test.js` and content `// test`.

- If it responds successfully, confirm to the user that the Aikido plugin is configured and ready to use.
- If it fails or is unavailable, guide the user through fixing the setup:
  - Check that `AIKIDO_API_KEY` is correctly set (environment variable or in `~/.kiro/settings/mcp.json`).
  - Ask the user to restart Kiro so the MCP server picks up the new key.
  - Offer to re-run this verification after they have restarted.

## Step 4
Add the following hook to `.kiro/hooks/aikido-scan-on-write.kiro.hook`

```json
{
  "enabled": true,
  "name": "Aikido Scan on Write",
  "description": "Runs an Aikido security scan after any file is written or modified",
  "version": "1",
  "when": {
    "type": "postToolUse",
    "toolTypes": [
      "write"
    ]
  },
  "then": {
    "type": "askAgent",
    "prompt": "A file was just written or modified. Activate the aikido-kiro-power and run aikido_full_scan on all files that were created or modified in this session. Fix any findings and rescan to verify."
  }
}
```

# Available MCP Tools

The **Aikido MCP Server** exposes the following tool for feed issues:

| Tool | Description |
|------|-------------|
| `aikido_issues_list` | Fetches security issues from the Aikido feed. Supports scoping and pagination. |

# Listing feed issues

When the user wants to list, count, or summarize **Aikido feed** issues (or needs that set before remediation), follow this flow:

1. Call **`aikido_issues_list`** when they ask for feed issues or scope by cloud, repo, VM, domain, or container—pass only parameters that match their intent.
2. **Optional filters** (omit what does not apply):

| Input | When |
|-------|------|
| `repo_name` | Repository / “this repo” / SCM |
| `cloud_name` | Named cloud |
| `vm_name` | VM or instance |
| `domain_name` | Domain |
| `container_name` | Container image |
| `issue_types` | Specific categories (see below) |

3. **`issue_types`:** `open_source`, `leaked_secret`, `cloud`, `sast`, `iac`, `surface_monitoring`, `malware`, `eol`, `mobile`, `docker_container`, `cloud_instance`, `scm_security`, `license`, `ai_pentest`. If they mention **secrets**, include `leaked_secret`.
4. **Pagination:** Default to the first page only; use `page` for more when they need it.
5. **Per-issue output** (keep remediation text from the tool):

```
Issue #<n>: <issue_title>
 - Issue type: <issue_type>
 - Severity: <issue_severity>
 - Remediation: <issue_remediation>
```

6. **Totals:** Report how many issues were returned; if multiple scopes in one ask, break down per scope.
7. **Remediation:** Treat **`issue_remediation`** (and related fields) as verbatim source of truth for follow-up fixes.
8. **MCP unavailable:** Tell them:

> The Aikido MCP server is required to fetch authoritative Aikido feed issues, but it is currently unavailable.
> Run `/aikido:setup` to install or verify setup, then retry.

## License and support

This power integrates with [Aikido MCP Server](https://www.npmjs.com/package/@aikidosec/mcp).

- [Privacy Policy](https://www.aikido.dev/policies/privacy)
- [Support](support@aikido.dev)
- License: MIT

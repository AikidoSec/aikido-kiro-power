---
name: "aikido-security-scan"
displayName: "Aikido Security Scan"
description: "Brings Aikido Security scanning directly into Kiro. Detects SAST vulnerabilities, exposed secrets, and IaC misconfigurations in code you write or modify, and guides Kiro to fix them before they ship."
keywords: ["aikido", "security", "iac", "sast", "secrets", "vulnerabilities", "scan", "issues", "code", "file", "write", "generate"]
author: Aikido Security
---

# Onboarding
Before proceeding, validate that the user has completed the following steps:

## Step 1: Verify Node.js version

Before doing anything else, run `node --version` to check the installed Node.js version.

- If Node.js is not installed or the command fails, stop and tell the user that Node.js 18.19.0 or higher is required to run the Aikido MCP server, and direct them to [nodejs.org](https://nodejs.org) to install it.
- If the version is below 18.19.0, stop and tell the user that the Aikido MCP server requires Node.js 18.19.0 or higher, show the currently installed version, and direct them to [nodejs.org](https://nodejs.org) to upgrade.
- If the version is 18.19.0 or higher, proceed with the steps below.

## Step 2: Sign in to Aikido

Authentication uses a browser-based sign-in flow via the **aikido_login** tool

**If the user is asking to switch accounts or re-authenticate**, call **aikido_login** with `force_reauth: true` and skip step 1 in the setup flow — the tool will always return fresh sign-in URLs. Continue from step 3 to walk the user through opening the URL and step 4 to verify within the setup flow.

**Otherwise**, follow the standard setup flow:

1. Check the MCP server is reachable and the user is signed in by calling **aikido_login** with no arguments. The tool is idempotent — it returns "Already signed in" if a valid token is cached, otherwise it starts a new sign-in flow.
2. If it reports the user is already signed in, confirm to the user that the Aikido plugin is configured and ready to use. Stop here.
3. If it returns region-specific sign-in URLs (EU / US / ME), present them to the user verbatim — do not strip or modify the `state` or `redirect_uri` query parameters — and ask them to open the URL for their region in a browser to complete sign-in.
4. Once the user reports they have completed the browser sign-in, verify the setup by calling **aikido_login** again.
5. If the **aikido_login** call itself fails because the MCP server is unavailable, tell the user to ensure the Aikido plugin is installed and that Kiro has loaded the `aikido` MCP server, then ask them to restart Kiro and retry.

## Step 3
Add the following hook to `.kiro/hooks/aikido-scan-on-write.kiro.hook`

\`\`\`json
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
\`\`\`

# Available MCP Tools

The **Aikido MCP Server** provides the following tools:

| Tool | Description |
|------|---------|
| `aikido_login` | Signs the user in through a browser-based flow and caches the auth token. Idempotent — returns "Already signed in" when a valid token exists. Pass `force_reauth: true` to switch accounts or re-authenticate. |
| `aikido_full_scan` | Runs an Aikido SAST, IaC and Secrets scan locally on provided code files. |
| `aikido_issues_list` | Lists, counts, and summarizes security issues from the Aikido security feed for the connected workspace — including open source (SCA), leaked secrets, SAST, IaC, cloud, container, EOL, license, malware, and surface monitoring findings. |

# Auto-scan after code generation

Automatically run a security scan after every response in which you generate, add, or modify one or more code files — without waiting for the user to ask.

1. After writing or modifying any code files, collect all affected file paths.
2. For each file, read its full content.
3. Run **aikido_full_scan** using the **aikido** MCP server with all files and their full content. Stay within the 50-file limit per request — batch into multiple calls if needed.
4. If any security issues are found:
   - Explain each issue clearly: title, description, severity, file location, and line numbers.
   - Apply fixes guided by the remediation provided by Aikido.
   - After applying all fixes, run **aikido_full_scan** again to verify that the issues were resolved and no new issues were introduced.
   - **Stopping the loop:** If you can explain why the applied fix is safe (e.g. the fix correctly addresses the finding and the remaining scan output is a false positive or acceptable), you may stop and report to the user. Otherwise, repeat the fix-and-rescan cycle up to 3 attempts; if issues remain after that, report them to the user instead of continuing.
5. Report the final scan result to the user — confirm all clear or list any unresolved issues with explanation.

# Best practices

When scanning the code for security vulnerabilities using the Aikido MCP server:

1. Identify all files that were generated, added, or modified in this session (or that the user has mentioned).
2. For each file, read its full content.
3. Run **aikido_full_scan** using the **aikido** MCP server with all files and their full content. Stay within the 50-file limit per request — batch into multiple calls if needed.
4. If any security issues are found:
   - Explain each issue clearly: title, description, severity, file location, and line numbers.
   - Apply fixes guided by the remediation provided by Aikido.
   - After applying all fixes, run **aikido_full_scan** again to verify that the issues were resolved and no new issues were introduced.
   - **Stopping the loop:** If you can explain why the applied fix is safe (e.g. the fix correctly addresses the finding and the remaining scan output is a false positive or acceptable), you may stop and report to the user. Otherwise, repeat the fix-and-rescan cycle up to 3 attempts; if issues remain after that, report them to the user instead of continuing.
5. Report the final scan result to the user — confirm all clear or list any unresolved issues with explanation.

# Listing Aikido feed issues

When the user wants to list, count, summarize, or triage the security issues already surfaced by Aikido for their workspace (vulnerabilities, leaked secrets, SAST/IaC/SCA results, cloud or container findings, EOL/license/malware alerts):

1. Use **aikido_issues_list** via the **aikido** MCP server.
2. Call it whenever the user wants to list, show, count, or summarize Aikido feed issues.
3. Pass scope fields only when the user (or workspace context) supplies them: `cloud_name`, `repo_name`, `vm_name`, `domain_name`, `container_name`.
4. Optional `issue_types` (array): `open_source`, `leaked_secret`, `cloud`, `sast`, `iac`, `surface_monitoring`, `malware`, `eol`, `mobile`, `docker_container`, `cloud_instance`, `scm_security`, `license`, `ai_pentest` — e.g. include `leaked_secret` for secrets. Omit when no category filter is needed.
5. Pagination: use the numeric `page` field only when the user needs more than the first page of results (zero-indexed). Only 25 findings are reported per page. Report to the user if there are more findings on following pages.
6. Present each issue exactly in this form (increment `#`):

```text
Issue #1: <issue_title>
 - Issue type: <issue_type>
 - Severity: <issue_severity>
 - Remediation: <issue_remediation>
```

If the Aikido MCP server is not available or fails, inform the user that the Aikido MCP server is required for Aikido feed issues but is not available, and ask them to complete the onboarding steps above (or re-run **aikido_login**) before retrying.

# Common Workflows

### Workflow 1: Auto-scan code you write or modify

**Goal:** Catch SAST vulnerabilities, exposed secrets, and IaC misconfigurations in code before it ships.

**Steps:**

1. After generating, adding, or modifying any code files, collect all affected file paths.
2. Read the full content of each affected file.
3. Run **aikido_full_scan** with all files and their full content (batch into multiple calls to stay within the 50-file limit per request).
4. If issues are found, explain each one (title, description, severity, file location, line numbers), apply the remediation guidance, and rescan to verify.
5. Report the final result — confirm all clear or list any unresolved issues with explanation.

**Example:**

```text
Agent: "I just modified auth/login.js. Running an Aikido scan to check for security issues."

[Uses aikido_full_scan with the affected files and their full content]

Agent: "Aikido flagged a hardcoded secret on line 12. Here's the fix using an
environment variable, and the rescan now comes back clean."
```

### Workflow 2: List and triage Aikido feed issues

**Goal:** Review, count, or summarize the open security findings already surfaced by Aikido for the workspace.

**Steps:**

1. Run **aikido_issues_list** when the user wants to list, show, count, or summarize Aikido feed issues.
2. Pass scope fields only when the user (or workspace context) supplies them: `cloud_name`, `repo_name`, `vm_name`, `domain_name`, `container_name`.
3. Optionally filter with `issue_types` (array): `open_source`, `leaked_secret`, `cloud`, `sast`, `iac`, `surface_monitoring`, `malware`, `eol`, `mobile`, `docker_container`, `cloud_instance`, `scm_security`, `license`, `ai_pentest` — e.g. include `leaked_secret` for secrets. Omit when no category filter is needed.
4. For pagination, use the numeric `page` field (zero-indexed) only when the user needs more than the first page. Only 25 findings are reported per page — tell the user if more findings exist on following pages.
5. Present each issue exactly in this form (increment `#`):

```text
Issue #1: <issue_title>
 - Issue type: <issue_type>
 - Severity: <issue_severity>
 - Remediation: <issue_remediation>
```

**Example:**

```text
User: "Show me the leaked secrets Aikido found in the payments repo."

[Uses aikido_issues_list with repo_name: 'payments' and issue_types: ['leaked_secret']]

Agent: "Issue #1: AWS access key committed to source
 - Issue type: leaked_secret
 - Severity: critical
 - Remediation: Revoke the key and move it to a secret manager..."
```

If the Aikido MCP server is not available or fails, inform the user that the Aikido MCP server is required for feed issues but is not available, and ask them to complete the onboarding steps above (or re-run **aikido_login**) before retrying.

## License and support

This power integrates with [Aikido MCP Server](https://www.npmjs.com/package/@aikidosec/mcp).
- [Privacy Policy](https://www.aikido.dev/policies/privacy)
- [Support](support@aikido.dev)
- License: MIT


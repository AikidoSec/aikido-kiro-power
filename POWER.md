---
name: "aikido-security-scan"
displayName: "Scan code with Aikido Security"
description: "Brings Aikido Security scanning directly into Kiro. Detects SAST vulnerabilities, exposed secrets, and IaC misconfigurations in code you write or modify, and guides Kiro to fix them before they ship."
keywords: ["aikido", "security", "iac", "sast", "secrets", "scan", "vulnerabilities", "scan", "issues", "code", "file", "write", "generate"]
author: Aikido Security
---

# Onboarding
Before proceeding, validate that the user has completed the following steps:

## Step 1

Prompt the user to configure their own Aikido API key for authentication. They can get Aikido API key at [app.aikido.dev](https://app.aikido.dev/settings/integrations/ide/kiro) → Settings → Integrations → IDE Plugins.
Once they have their API key they can set it as an environment variable named AIKIDO_API_KEY on their system, or hardcode it directly into the user level MCP configuration file (usually at ~/.kiro/settings/mcp.json) in the power section. Guide the user on how to do this.

## Step 2

Validate user has Node.js 18+, required to run the Aikido MCP server

## Step 3

Verify the MCP server is reachable and the API key is valid by calling **aikido_full_scan** with a minimal test payload: one file with path `test.js` and content `// test`.

- If it responds successfully, confirm to the user that the Aikido plugin is configured and ready to use.
- If it fails or is unavailable, guide the user through fixing the setup:
  - Check that `AIKIDO_API_KEY` is correctly set (environment variable or in `~/.kiro/settings/mcp.json`).
  - Ask the user to restart Kiro so the MCP server picks up the new key.
  - Offer to re-run this verification after they have restarted.

## Step 4
Add a hook to `.kiro/hooks/aikido-scan-on-write.kiro.hook`

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
| `aikido_full_scan` | Runs an Aikido SAST, IaC and Secrets scan locally on provided code files. |

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
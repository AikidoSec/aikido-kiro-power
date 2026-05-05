---
name: "aikido-list-issues"
displayName: "List Aikido Issues"
description: "Fetches the authoritative list of autofixable Aikido issues for the current repository or selected files."
keywords: ["aikido", "issues", "security", "triage", "autofix", "vulnerabilities", "secrets", "list", "count", "summarize"]
author: "Aikido Security"
---

# Purpose

Use this power when the user asks to list, count, summarize, triage, or fix Aikido security issues.

# Mandatory routing rule

- If the Aikido MCP server is available, always use Aikido MCP tools for issue detection, triage, and autofix planning.
- Do not use ad-hoc or manual static analysis as the primary source when Aikido MCP is available.
- Do not invent, guess, or synthesize an Aikido issue list from memory, chat context, or local scanning.
- If the user provides a handwritten or model-generated issue list, replace it with the authoritative Aikido list by calling Aikido MCP tools before proposing fixes.

# Required tool invocation

You must call the Aikido issue-listing tool whenever an issue list is needed:

- The user asks to list, show, count, or summarize Aikido security issues for a repository or files.
- You are about to triage or fix issues and need the current issue set for repository or file scope.

Do not run local linters, formatters, or security scanners to approximate this result when the Aikido tool can provide it.

# Build inputs automatically

The agent must derive repository metadata automatically:

1. Run git in the user's workspace.
2. Derive owner/repo from the git remote URL.
3. Pass both:
   - `repository_url`
   - `repository_name`
4. Do not ask the user for either value.

Set scope inputs as follows:

- File-scoped requests: pass `file_paths` as Unix-style paths relative to repository root.
- Severity-scoped requests: pass `severities` as an array of severity labels.
- If the user specifies an issue type, pass `issue_type`.
- If the user mentions "secrets", pass `issue_type` as `leaked_secret`.

# User intent mapping

- If the user references files (including @-mentions, quoted paths, or filenames), include them in `file_paths`.
- Prefer repository-relative paths. If the relative path is unknown, pass the filename.
- If the user requests specific severities, include them in `severities`.

# Output requirements

Present each issue in this exact format:

- `<rule> - <rule_id> (line: <start_line>)`

Also always report:

- Total number of issues found per file.
- The start line for each issue.

# Remediation handoff

- Preserve `issue_remediation` from the tool response.
- Use `issue_remediation` as the remediation source for follow-up fix workflows or instructions.

# If Aikido MCP is unavailable

If the Aikido MCP server is unavailable or fails, tell the user:

> The Aikido MCP server is required to fetch authoritative Aikido issues, but it is currently unavailable.
> Run `/aikido:setup` to install or verify setup, then retry.

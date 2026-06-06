# Composio evaluation for Magnus / Infinity Drift

## What Composio appears to provide

Composio is an integration layer for AI agents. It exposes many external services as tools/toolkits and handles parts of auth/OAuth, user connections, triggers, and execution. The docs describe three practical usage modes:

- CLI: local `composio` command for search/connect/execute/proxy/run workflows.
- SDK/native tools: explicit tool schemas passed to model/framework.
- MCP: expose tools through Model Context Protocol to MCP-compatible clients.

Composio marketing/docs emphasize 1000+ app/toolkits, per-user auth/context, triggers, and a workbench/sandbox for programmatic execution.

## Recommended first path for Hermes

Start with **CLI proof of concept**, not a broad MCP integration.

Reasoning:

- CLI lets Hermes test one toolkit at a time via terminal without changing the global tool surface.
- It is easier to inspect exact commands, outputs, and auth state.
- It avoids loading hundreds/thousands of tools into the model context.
- It is safer while approval boundaries are still being designed.

Candidate install/login flow from Composio docs:

```bash
curl -fsSL https://composio.dev/install | bash
composio login
composio status
```

Then for a single toolkit:

```bash
composio search "<what the agent needs>"
composio link <toolkit>
composio execute <TOOL_SLUG> '<json args>'
```

Do not run install/login automatically unless Magnus asks to proceed.

## Best first toolkits for Magnus

1. **Notion**
   - Product docs, decision logs, roadmap/status.
   - Start with reading and creating pages in a dedicated `Hermes AI Inbox`.

2. **GitHub**
   - Read issues/PRs/repos; create AI-labeled suggestions after approval or in a sandbox repo.
   - No merge/admin/workflow dispatch initially.

3. **Airtable**
   - High value for Teoria content QA, question-pool analysis, metadata gaps, duplicate detection.
   - Start read-only; write to a separate AI QA table/view or require approval.

Google Drive is already covered outside Composio by the `google-drive-deliverables` skill, using `Dokumenter Hermes`.

## Integrations to delay

- AWS/Ovio production: design separate narrow IAM roles; do not grant broad broker access.
- Payments, finance, Visma, Vipps/MobilePay, Kravia/inkasso: report/read-only first, no writes without explicit approval.
- Email/Slack/Teams sending: drafts/summaries are useful, but posting/sending requires explicit approval.

## Approval model

Use the four access classes from `agent-tool-integrations`:

- Class A read: proceed when scope is clear.
- Class B create internal artifacts: allowed when requested.
- Class C modify existing data: explicit approval.
- Class D external/irreversible/high-impact: explicit approval every time.

## MCP note

Hermes supports native MCP through `mcp_servers` in `~/.hermes/config.yaml`. Composio can expose tools via MCP, but this should be tested only after the useful toolkits are known. Broad MCP tool surfaces can increase context cost and make tool choice noisier. If MCP is used, keep the server/toolkit set narrow and disable unneeded tools where possible.

## Production hardening idea

If Composio becomes important, build a thin Hermes plugin/wrapper rather than relying only on raw CLI/MCP. The wrapper should:

- allowlist specific toolkits/actions
- require approval for Class C/D actions
- log external mutations with timestamp, toolkit, action, arguments summary, and result link/ID
- redact secrets and PII-heavy outputs
- cap output sizes
- route created documents to approved destinations such as `Dokumenter Hermes` or `Hermes AI Inbox`

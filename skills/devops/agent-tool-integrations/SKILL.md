---
name: agent-tool-integrations
description: "Evaluate and connect external tool-integration brokers (Composio, MCP servers, CLI bridges, API aggregators) to Hermes or similar agents with least-privilege, approval boundaries, and staged rollout."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [macos, linux]
metadata:
  hermes:
    tags: [agent-integrations, composio, mcp, oauth, security, tool-brokers]
    related_skills: [hermes-agent, google-drive-deliverables, gateway-credential-setup]
    created_by: agent
---

# Agent Tool Integrations

Use this skill when Magnus asks whether/how to connect an agent to many external services through an integration broker such as Composio, an MCP server, a CLI bridge, OAuth gateway, or API aggregation layer.

The goal is not merely to "connect more tools". The goal is to increase useful operational capability while preserving drift control, auditability, least privilege, and explicit approval for external or irreversible actions.

## Core recommendation

Do **not** start by connecting everything. Roll out integration brokers in stages:

1. **Read-first, low-risk services**: context gathering, search, summaries, reports.
2. **Internal artifact creation**: create reports/pages/issues in a clearly marked AI inbox or Drive folder.
3. **Controlled writes**: modify existing operational data only after explicit approval.
4. **High-risk actions**: deploys, AWS production actions, email/posting, payments, finance, inkasso, and deletion require explicit approval and narrow technical controls.

For Magnus / Infinity Drift, prefer **direct, one-service-at-a-time integrations** over broad aggregators when the connected systems are sensitive. Composio-style brokers are useful for exploration, but if the broker is compromised it can become a single high-blast-radius point across many services. Default to native skills/tokens for Notion, Airtable, GitHub, Google Drive, etc.; only introduce an aggregator when its value clearly exceeds the added concentration risk.

## Access classes

Classify every connected toolkit/action before use:

- **Class A — Read access**: can usually proceed when task scope is clear.
  - Examples: list GitHub issues, read Notion pages, read Airtable records, inspect deployment status.
- **Class B — Create internal artifacts**: allowed when Magnus asked for a deliverable.
  - Examples: upload a report to `Dokumenter Hermes`, create an AI Inbox page, create an issue draft/AI suggestion.
- **Class C — Modify existing data**: requires explicit approval.
  - Examples: update Airtable content, edit Notion product docs, change issue status/priority, change deployment config.
- **Class D — External/irreversible/high-impact actions**: always require explicit approval and ideally technical guardrails.
  - Examples: send email/messages, share documents, delete files, merge PRs, trigger prod deploy/rollback, AWS prod actions, payment/finance/inkasso operations.

## Recommended rollout for Magnus / Infinity Drift

Start with at most 2–3 integrations:

1. **Notion** — product context, roadmap/status, decision logs, AI Inbox pages.
2. **GitHub** — read issues/PRs/repos first; write only AI-labeled suggestions unless approved.
3. **Airtable** — especially Teoria QA/reporting; read-first, write only after approval.

Google Drive deliverables are already covered by `google-drive-deliverables`; use Composio for Google only if native Docs/Sheets/Gmail functionality is needed beyond Drive file upload.

Delay or isolate:

- **AWS/Ovio prod**: never broad broker access. If needed later, use a narrow IAM role with read-only CloudWatch/ECS or one audited action such as `ecs force-new-deployment` for the specific cluster/service.
- **Payments/finance/inkasso**: read/report first; no write actions without explicit approval.
- **Email/Slack/Teams**: summaries/drafts are useful, but sending/posting always requires approval.

## Preferred technical path

### Direct service integrations (preferred for Magnus)

Use direct service-specific credentials/skills when the service contains Infinity Drift production, product, content, finance, support, or source-code data.

Recommended pattern:

1. Store credentials in `~/.hermes/.env` with explicit names; never paste secrets into chat if a local file/edit path is available.
   - For Hermes itself, use `hermes config env-path` to locate the active env file, then set secrets there.
   - Do **not** use `hermes config set GITHUB_TOKEN ...` or similar for secrets; `hermes config set` writes structured Hermes settings to `config.yaml`, not runtime credentials.
2. Verify auth with harmless read-only API calls that return account/workspace/repo/base metadata, not sensitive business content.
   - For GitHub tokens intended for Hermes tools/subagents, verify with `GH_TOKEN`/`GITHUB_TOKEN` in the command environment and a read-only API call such as repo metadata or `gh auth status`, without requiring a persistent `gh auth login` session.
3. Record only non-secret identifiers, scopes, and visible resources in a reference file.
4. Treat tokens with write/admin scopes as **operationally read-only** until Magnus explicitly approves each write/destructive/external action.
5. If multiple accounts exist for the same provider, use separate env vars and explicitly select the token per operation (e.g. default GitHub vs Infinity Drift GitHub).

### Broker / CLI bridge proof of concept

1. Install and login with the broker CLI.
2. Search/connect one toolkit.
3. Execute harmless read-only calls.
4. Record exact commands and outputs.

### Operational skill

Once useful commands are verified, add a reference file under this skill documenting allowed toolkits, commands, and approval boundaries.

### MCP only after the tool surface is understood

Hermes supports native MCP, but broad MCP servers can expose many tools and increase context/tool-call risk. Prefer small, scoped MCP servers or explicit tool allowlists when possible.

### Custom Hermes plugin/wrapper for production use

If a broker becomes central, build a thin wrapper that enforces allowlists, approval prompts, logging, output limits, and destructive-action blocks.

## Workflow checklist

When evaluating a new broker/toolkit:

1. Identify business use case and expected value.
2. List services and actions needed.
3. Assign each action to Class A/B/C/D.
4. Start with read-only or sandbox/test workspace where possible.
5. Verify authentication without exposing tokens/secrets.
6. Run one harmless read-only command and save exact command/output summary in `references/`.
7. Add approval boundaries before enabling writes.
8. For recurring use, create a stable skill/reference instead of relying on ad-hoc memory.

## Pitfalls

- Do not mistake OAuth success for safe authorization design. Broad tokens need operational constraints.
- Do not connect AWS, payments, email sending, or production deployment actions as a first test.
- Do not let an MCP server/toolkit flood the agent context with hundreds of tools unless there is a clear need.
- Do not post, send, share, delete, deploy, merge, or mutate business-critical data without explicit approval.
- Do not create a one-service sprawl of narrow skills; add concise references under this umbrella for each broker/toolkit evaluation.

## References

- `references/composio-evaluation.md` — Composio-specific assessment and staged adoption plan from the Magnus/Infinity Drift context.
- `references/magnus-direct-service-access.md` — Direct Notion/Airtable/GitHub/Drive setup and verification pattern for Magnus, including access-class boundaries and known non-secret resource identifiers.
- `references/hermes-github-token-setup.md` — Hermes-specific pattern for placing GitHub tokens in the active `.env`, not `config.yaml`, and verifying with read-only GitHub metadata calls.

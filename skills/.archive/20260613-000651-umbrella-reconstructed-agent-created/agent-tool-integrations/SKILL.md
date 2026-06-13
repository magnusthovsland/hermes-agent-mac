---
name: agent-tool-integrations
description: Evaluate and connect external tool-integration brokers (Composio, MCP servers, CLI bridges, API aggregators) to Hermes or similar agents with least-privilege, approval boundaries, and staged rollout.
version: 1.0.0
author: Hermes Agent
license: MIT
---

# Agent Tool Integrations (Archived)

This narrow operational skill was consolidated into `hermes-agent` under "External tool integrations and brokers".

Core preserved rule: do not connect everything at once. Roll out read-first services, then internal artifact creation, then controlled writes, then high-risk actions only with explicit approval and narrow scopes. Prefer direct one-service-at-a-time integrations for sensitive systems; use brokers/MCP/CLI bridges only after tool surface, auth, logging, and approval boundaries are understood.

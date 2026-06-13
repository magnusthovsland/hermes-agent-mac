---
name: gateway-credential-setup
description: Set up messaging gateway credentials for Hermes or similar agent gateways without reusing legacy tokens or confusing templates with active secrets.
version: 1.0.0
author: Hermes Agent
license: MIT
---

# Gateway Credential Setup (Archived)

This narrow operational skill was consolidated into `hermes-agent` under "Gateway credential setup".

Core preserved rule: before saying a platform is configured, distinguish (1) template/commented env lines, (2) active runtime environment variables, and (3) proof that the active credential belongs to the intended bot/platform identity. Do not reuse legacy OpenClaw/bot tokens for a fresh Hermes setup unless explicitly requested. Report secrets as SET/not set only, restart gateway after changes, then verify with platform status/logs and a real message route.

Original package had Telegram-focused references about allowlist debugging, fresh-bot setup, and single-user hardening; those concepts are now summarized in `hermes-agent`.

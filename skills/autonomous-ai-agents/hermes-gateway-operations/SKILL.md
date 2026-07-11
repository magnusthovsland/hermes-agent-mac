---
name: hermes-gateway-operations
description: "Operate and verify Hermes messaging gateways after updates/config changes, especially from gateway-hosted sessions where self-restart is guarded."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, gateway, telegram, update, restart, operations, verification]
    created_by: agent
---

# Hermes Gateway Operations

Use this skill when working on Hermes messaging gateway runtime state: updating Hermes from Telegram/Discord/etc., restarting the gateway, verifying that the live bot picked up new code/config, or diagnosing gateway-service status.

This is a user-local operational companion to the bundled `hermes-agent` skill. The bundled skill remains the source of truth for general Hermes CLI references; this skill captures local operational workflow lessons that are useful during gateway-hosted sessions.

## Core workflow

1. **Check current state before changes**
   ```bash
   which hermes
   hermes --version
   hermes status --all
   hermes config path
   ```

2. **Run the requested Hermes update**
   ```bash
   hermes update
   ```

3. **Verify the installed code, not just the updater output**
   ```bash
   hermes --version
   hermes status --all
   ```

4. **Verify requested capability with a real execution path**
   - For model availability, run a minimal `hermes chat` invocation against the exact provider/model.
   - Example:
     ```bash
     hermes chat -Q --provider openai-codex -m gpt-5.6-sol -q 'Svar kun med ordet OK hvis du kan svare.'
     ```
   - Treat a successful model response as stronger evidence than catalog/source-code matches alone.

5. **Restart or reload the gateway if live bot runtime must pick up new code/config**
   - Code/config changes may be installed on disk while the gateway process is still running old code.
   - Verify gateway status after restart:
     ```bash
     hermes gateway status
     ```

## Gateway self-restart guard

When running inside a gateway-hosted agent session, `hermes gateway restart` may be refused with a message like:

```text
Refusing to restart the gateway from inside the gateway process.
This command was blocked to prevent restart loops.
Use `hermes gateway restart` from a shell outside the running gateway.
```

This is a safety guard, not a failed update.

### What to do instead

- **Do not claim the live bot has restarted** if the guard blocked the command.
- Tell the user clearly that the installed code is updated, but the gateway process still needs an external restart.
- Recommended user-facing instruction:
  ```bash
  hermes gateway restart
  ```
  run from a normal terminal outside the gateway process.
- If an approved platform slash command exists for the gateway, `/restart` may be an option, but verify the command actually succeeded before saying the live bot is on the new runtime.

## Model availability checks after update

For provider/model checks, use layered evidence:

1. **Installed Hermes version is up to date** (`hermes --version`).
2. **Catalog/source has model metadata** when relevant.
3. **Real model call succeeds** with the exact provider and model.

Report the exact command shape and actual response. Keep the final answer short and distinguish:

- **Installed code updated**
- **Capability verified**
- **Gateway runtime restarted or still pending external restart**

## Pitfalls

- Do not treat `hermes update` success as proof the Telegram/Discord gateway process is already using the new code.
- Do not hide update conflicts or partial warnings; report them if they affect the requested outcome.
- If local changes were stashed/restored during update, mention only durable/actionable consequences. Avoid turning transient conflict text into persistent claims unless it blocks the task.
- Avoid editing protected bundled skills. Capture local operational lessons here instead.

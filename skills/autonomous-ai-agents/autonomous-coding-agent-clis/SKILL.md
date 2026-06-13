---
name: autonomous-coding-agent-clis
description: "Class-level umbrella for orchestrating external autonomous coding CLIs such as Claude Code, Codex, and OpenCode from Hermes terminal/process tools."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [autonomous-agents, coding-agents, claude-code, codex, opencode, tmux, worktrees]
    related_skills: [hermes-agent, github-workflows]
---

# Autonomous Coding Agent CLIs

## Overview
Use this umbrella when delegating software work to a standalone coding-agent CLI. The common class is: Hermes remains the orchestrator, while an external process edits code, runs tests, reviews PRs, or performs long-running implementation work.

## When to Use
- The user explicitly asks to use Claude Code, Codex, or OpenCode.
- A coding task is large enough to benefit from an external autonomous worker.
- You need parallel workstreams in separate worktrees or persistent TUI sessions.
- You want a PR review/implementation pass from a second agent.

Do not spawn an external CLI when the built-in `delegate_task` tool is sufficient, when prerequisites/auth are missing, or when direct editing is faster and safer.

## Common Orchestration Rules
1. Verify the binary and auth status before assigning work.
2. Prefer one-shot/non-interactive modes for bounded tasks; use tmux/PTY only for long-running interactive sessions.
3. Run inside a git repository; use separate worktrees for parallel edits.
4. Capture verifiable outputs: changed files, test results, PR URL, branch name, or review text.
5. For background jobs, use `terminal(background=true, notify_on_complete=true)` or track via `process`.
6. Do not claim success from an agent’s self-report alone; inspect files/git/tests yourself.

## CLI-Specific Notes

### Claude Code
Install/auth:
```bash
npm install -g @anthropic-ai/claude-code
claude auth status
claude doctor
claude --version
```
Prefer print mode for most bounded work:
```bash
claude -p "Implement X, run tests, summarize changed files"
```
Use interactive PTY/tmux when a multi-turn coding session is needed. Handle first-run trust and permission dialogs explicitly; do not assume the TUI is ready until captured output proves it.

### OpenAI Codex CLI
Install/auth:
```bash
npm install -g @openai/codex
codex --version
```
Codex generally expects a git repository and often needs `pty=true` for interactive use. Hermes’ OpenAI Codex provider credentials are not automatically proof that the standalone CLI is authenticated; CLI OAuth may live under `~/.codex/auth.json`.

Typical one-shot:
```bash
codex exec "Add dark mode toggle to settings"
```
For scratch tasks, initialize a temporary git repo first because Codex may refuse to run outside one.

### OpenCode
Install/auth:
```bash
npm i -g opencode-ai@latest
# or: brew install anomalyco/tap/opencode
opencode auth list
opencode --version
```
Resolve binary ambiguity before debugging behavior differences:
```bash
which -a opencode
opencode --version
```
Pin an explicit binary path if Hermes resolves a different executable than the user’s shell.

## Patterns

### One-shot implementation
```bash
terminal(command="<agent> <one-shot-command> 'Implement the requested change and run the targeted tests'", workdir="/path/to/repo", timeout=600)
```
Inspect `git diff`, run tests yourself, and summarize concrete results.

### Long-running interactive session
```bash
tmux new-session -d -s coding-agent -x 120 -y 40 '<agent command>'
tmux send-keys -t coding-agent 'Task prompt...' Enter
tmux capture-pane -t coding-agent -p
```
Use `process`/tmux captures for progress checks. Close the session when done.

### Parallel issue fixing
Create one worktree per issue/task, spawn one agent per worktree, then reconcile diffs and tests before merging.

## Pitfalls
- Confusing Hermes provider auth with standalone CLI auth.
- Letting a CLI modify the main worktree while you make concurrent edits.
- Trusting a TUI screen that is waiting on a hidden prompt/dialog.
- Forgetting to verify changed files and tests after the agent exits.
- Leaving background/tmux sessions running after the task is complete.

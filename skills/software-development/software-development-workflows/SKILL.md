---
name: software-development-workflows
description: "Umbrella workflow for software engineering tasks: spikes, TDD, debugging, code review, simplification, codebase inspection, and language/runtime debuggers."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [software-development, debugging, testing, tdd, code-review, refactoring, codebase-inspection]
---

# Software Development Workflows

Use this skill for class-level software work: understanding a codebase, proving feasibility with a spike, writing tests first, debugging root causes, reviewing quality/security, simplifying recent changes, or attaching language/runtime debuggers.

## Triage: choose the mode

- **Unknown codebase or scope** → inspect structure, languages, LOC, dependencies, and test commands first.
- **Unproven approach** → run a time-boxed spike in an isolated scratch path; preserve only the learning.
- **Feature or bug with clear behavior** → use TDD: write/observe failing test, implement the smallest fix, refactor after green.
- **Bug without root cause** → use systematic debugging; reproduce, localize, explain mechanism, then fix.
- **Pre-commit or PR readiness** → request/recreate a code review: diff, tests, lint, security scan, docs impact, and auto-fix safe issues.
- **Messy recent changes** → simplify code with parallel review perspectives if appropriate.
- **Runtime-specific failure** → use Python `pdb`/`debugpy` or Node `--inspect`/Chrome DevTools Protocol.
- **CMS/API access or content attribution question** → perform a read-only access/audit workflow: identify credential purpose without exposing secrets, query recent content, inspect transaction history, and map author IDs to users/robots.

## Operating rules

1. **Ground every claim in tools.** Read files, inspect git state, run tests, and capture exact failures.
2. **Do not skip reproduction.** A fix without a reproduced failure is a guess unless the user explicitly asks for a static patch only.
3. **Use the narrowest safe edit.** Avoid broad rewrites during debugging; refactor only after tests pass.
4. **Verify at the right level.** Run the smallest relevant test first, then expand to the package/project checks.
5. **Report durable evidence.** Include commands run and real pass/fail output in the final answer.

## Labeled playbooks

### Codebase inspection

Use `pygount` or language-native tools to quantify files, languages, generated/vendor ratios, and hotspots. The preserved package `references/packages/codebase-inspection/` contains the original commands and report style.

### Spike

A spike is disposable experimentation to reduce uncertainty. Keep it isolated, time-boxed, and conclude with what was learned, what failed, and whether to proceed.

### Test-driven development

Follow RED → GREEN → REFACTOR. Create the failing test before implementation when behavior is knowable. Do not broaden refactors until the targeted test is green.

### Systematic debugging

Use a four-phase loop: reproduce, localize, explain, fix/verify. Record hypotheses and falsification evidence; avoid patching symptoms.

### Code review and simplification

For pre-commit review, inspect diff and run quality gates. For simplification, ask what code can be deleted, what abstractions can collapse, and whether behavior remains covered by tests.

### CMS/API content access audits

When asked whether an agent has CMS access or who made recent content changes, keep the workflow read-only by default: find credential metadata, query current content, inspect transaction history for attribution, and map author IDs to humans/robots. See `references/sanity-content-lake-audit.md` for the Sanity Content Lake/History API pattern and reporting pitfalls.

### Python debugging

Use `pdb` for local interactive stepping and `debugpy` when a long-running process or test needs DAP-style attach. See `references/packages/python-debugpy/` for command recipes.

### Node debugging

Use `node --inspect` or `--inspect-brk`, connect to the Chrome DevTools Protocol, set breakpoints, evaluate expressions, and continue. See `references/packages/node-inspect-debugger/`.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `codebase-inspection`, `node-inspect-debugger`, `python-debugpy`, `requesting-code-review`, `simplify-code`, `spike`, `systematic-debugging`, and `test-driven-development`.

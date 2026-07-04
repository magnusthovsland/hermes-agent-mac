---
name: data-science-workflows
description: "Class-level umbrella for data-science workflows: iterative Python/Jupyter exploration, notebook-backed REPL sessions, dataframe/API inspection, and clean verification of computational notebooks."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [data-science, jupyter, notebook, repl, python, exploration, iterative]
---

# Data Science Workflows

Use this skill for exploratory or notebook-oriented computational work: iterative Python, dataframe inspection, API exploration, long-running analyses, notebook repair, and clean rerun verification. Prefer a live notebook/kernel workflow when state must persist across steps; prefer `execute_code` for one-shot scripts that need Hermes tool helpers; prefer `terminal` for package management, shell commands, builds, and process control.

## Tool Choice

| Workflow | Best tool | Why |
|---|---|---|
| Iterative exploration, dataframes, ML experiments, API poking | Live Jupyter kernel | Variables, imports, and objects persist across executions. |
| One-shot calculations or scripts that call Hermes tools | `execute_code` | Stateless but can call `web_search`, file ops, terminal wrappers, etc. |
| Installs, git, builds, services, shell pipelines | `terminal` | Native process management and environment control. |
| Final notebook validation | Restart + run all | Confirms reproducibility from a clean kernel. |

Rule of thumb: if a human would open a notebook for the task, use a notebook-backed workflow rather than repeatedly running isolated snippets.

## Live Jupyter Kernel Workflow (hamelnb)

The `hamelnb` helper gives Hermes a stateful Python REPL via a live Jupyter kernel.

### Prerequisites

1. `uv` must be installed (`which uv`).
2. JupyterLab must be available (`uv tool install jupyterlab` if missing).
3. A local Jupyter server must be running.

Helper script path:

```bash
SCRIPT="$HOME/.agent-skills/hamelnb/skills/jupyter-live-kernel/scripts/jupyter_live_kernel.py"
```

If the helper repository is not present:

```bash
git clone https://github.com/hamelsmu/hamelnb.git ~/.agent-skills/hamelnb
```

### Start or Find JupyterLab

```bash
uv run "$SCRIPT" servers --compact
```

If no server is available, start one for local agent access:

```bash
mkdir -p "$HOME/notebooks"
jupyter-lab --no-browser --port=8888 --notebook-dir=$HOME/notebooks \
  --IdentityProvider.token='' --ServerApp.password='' > /tmp/jupyter.log 2>&1 &
sleep 3
uv run "$SCRIPT" servers --compact
```

First execution after startup can time out while the kernel initializes; retry once before treating it as a failure.

### Create a Scratch Notebook Session

If no notebook exists, create/use `scratch.ipynb` and start a kernel session through Jupyter's REST API:

```bash
mkdir -p ~/notebooks
python - <<'PY'
import json, pathlib
p = pathlib.Path.home() / 'notebooks' / 'scratch.ipynb'
p.write_text(json.dumps({
  'cells': [{'cell_type': 'code', 'execution_count': None, 'metadata': {}, 'outputs': [], 'source': []}],
  'metadata': {'kernelspec': {'display_name': 'Python 3', 'language': 'python', 'name': 'python3'}},
  'nbformat': 4,
  'nbformat_minor': 5,
}), encoding='utf-8')
print(p)
PY
curl -s -X POST http://127.0.0.1:8888/api/sessions \
  -H "Content-Type: application/json" \
  -d '{"path":"scratch.ipynb","type":"notebook","name":"scratch.ipynb","kernel":{"name":"python3"}}'
```

### Execute Iteratively

Always use `--compact` to keep results concise.

```bash
uv run "$SCRIPT" notebooks --compact
uv run "$SCRIPT" execute --path scratch.ipynb --code 'import pandas as pd; print(pd.__version__)' --compact
```

For multiline code, use shell `$'...'` quoting:

```bash
uv run "$SCRIPT" execute --path scratch.ipynb --code $'import os\nfiles = os.listdir(".")\nprint(f"Found {len(files)} files")' --compact
```

State persists across calls. Use this to build objects gradually, inspect intermediate results, and retry small changes without recomputing all setup.

### Inspect Variables

Argument order matters: subcommand flags like `--path` go before the sub-subcommand.

```bash
uv run "$SCRIPT" variables --path scratch.ipynb list --compact
uv run "$SCRIPT" variables --path scratch.ipynb preview --name df --compact
```

### Edit Notebook Cells

```bash
uv run "$SCRIPT" contents --path scratch.ipynb --compact
uv run "$SCRIPT" edit --path scratch.ipynb insert --at-index 0 --cell-type code --source 'print("hello")' --compact
uv run "$SCRIPT" edit --path scratch.ipynb replace-source --cell-id <id> --source '<new code>' --compact
uv run "$SCRIPT" edit --path scratch.ipynb delete --cell-id <id> --compact
```

## Verification

When the user asks for a clean proof, or before presenting a notebook as finished, restart the kernel and run all cells:

```bash
uv run "$SCRIPT" restart-run-all --path scratch.ipynb --save-outputs --compact
```

Report the real output or traceback. Do not claim notebook reproducibility without a restart/run-all check.

## Practical Pitfalls

- The kernel Python is JupyterLab's Python. Install missing packages into that environment, not necessarily Hermes' environment.
- JSON outputs can be verbose; keep `--compact` on every helper command.
- If a session does not exist, execution fails even if the notebook file exists. Create a Jupyter session first.
- Errors are returned as structured JSON; inspect `ename`, `evalue`, and traceback fields.
- Increase timeout for heavy cells: `--timeout 120` or higher when appropriate.

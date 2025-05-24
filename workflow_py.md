# Python Workflow Implementation Overview

This document sketches how the CLI workflow described in `codex-old/workflow.md` can be realised in Python.

## Required Libraries
- **openai** – talk to the OpenAI API
- **typer** – build the command line interface
- **rich** – render a simple terminal chat UI (or use Textual for a full TUI)
- **python-dotenv** – load environment variables from `.env`
- **pytest** – testing framework

Install core dependencies with:
```bash
pip install openai typer rich python-dotenv
```

## Suggested Folder Layout
```
codex_py/
├── cli/
│   ├── __init__.py
│   └── main.py          # entry point for `codex`
├── agent/
│   ├── __init__.py
│   ├── loop.py          # equivalent to agent-loop.ts
│   └── exec.py          # run shell commands safely
├── ui/
│   ├── __init__.py
│   └── terminal.py      # Rich/Textual interface
├── utils/
│   ├── __init__.py
│   └── config.py        # read ~/.codex/instructions.md etc.
├── logs/                # rollout files
└── tests/
```
Expose `codex` as a console script in `pyproject.toml` pointing to `codex_py.cli.main:app`.

## Implementation Steps
1. **CLI Invocation**
   - Use Typer to parse flags and options. Load `.env` with `dotenv` and merge configuration files into a dataclass.
2. **Create UI and AgentLoop**
   - Instantiate `AgentLoop` with callbacks for streaming output and loading state. Render `ui.terminal` with Rich for interactive sessions. Quiet mode prints final output only.
3. **Send Prompt**
   - Convert the prompt to input items (simple dataclasses). Call `AgentLoop.run()` which makes a streaming request via `openai.ChatCompletion.create`.
4. **Process Responses**
   - Stream messages and function calls. Execute approved shell commands using `subprocess` or a Docker sandbox. Record output and feed it back as `function_call_output`.
5. **Loop & Finish**
   - Continue the exchange until the API replies with a completed status or the user cancels. Save the conversation JSON to `logs/` for later inspection.

This outline mirrors the TypeScript workflow while remaining idiomatic to Python.

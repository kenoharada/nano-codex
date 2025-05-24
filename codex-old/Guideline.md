# Guide: Rewriting the Codex CLI Project in Python

This document outlines a suggested approach for converting the existing TypeScript/Node.js project to Python. It covers a possible directory layout and recommended libraries. The goal is to replicate the current CLI features in a Python stack that is easy to understand for beginners.

## 1. Understand the Current Project

- The repository's main logic resides in **`codex-cli/`**. This folder contains the TypeScript sources that build the `codex` CLI tool.
- The CLI uses the **Ink** framework (React for the terminal) to render interactive components, and it talks to OpenAI's API through the `openai` Node library.
- Tests live under **`codex-cli/tests/`** and are written with **Vitest**.

Before rewriting, skim through these directories to familiarize yourself with the features to re‑implement.

## 2. Set Up a Python Environment

1. Install Python 3.11 or newer.
2. Create a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```
3. Install initial dependencies:
   ```bash
   pip install openai typer rich pytest
   ```
   - **Typer** builds friendly CLIs (based on Click).
   - **Rich** lets you print colored text and build simple text UIs. For more advanced terminal interfaces, consider **Textual**.

## 3. Suggested Folder Structure

```
codex_py/                # Root package
├── codex_cli/           # Python implementation of the CLI
│   ├── __init__.py
│   ├── main.py          # Entry point for `python -m codex_cli`
│   ├── app.py           # High level application logic
│   ├── agent/           # Agent loop and command execution
│   │   ├── __init__.py
│   │   ├── loop.py
│   │   └── sandbox.py
│   ├── utils/           # Helper modules
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── file_ops.py
│   │   └── parsing.py
│   └── components/      # Optional Rich/Textual UI pieces
│       ├── __init__.py
│       └── prompts.py
├── tests/               # Pytest test suite
│   └── test_basic.py
├── README.md
└── pyproject.toml       # Build configuration (alternatively setup.py)
```

- `main.py` exposes a `typer.Typer` application. Define commands and options that mirror the original Node CLI (e.g., `--model`, `--quiet`, `--approval-mode`).
- `agent/loop.py` should handle the interaction with the OpenAI API using the `openai` Python SDK. Stream responses and execute shell commands inside a sandbox (e.g., Docker or a restricted subprocess).
- `utils/config.py` reads configuration files from `~/.codex/` (similar to the TypeScript version).
- `tests/` contains pytest tests that validate key behaviours.

## 4. Implementation Steps

1. **Bootstrap the package**
   - Create the folder structure above and add an empty `__init__.py` file to each package.
   - Use `pyproject.toml` (with Poetry or plain setuptools) to manage dependencies.

2. **Recreate the CLI**
   - Translate the command‑line flags defined in `codex-cli/src/cli.tsx` to Typer options.
   - Start with a basic command that prints the received prompt.

3. **Integrate OpenAI**
   - Use the `openai` package to send prompts and stream responses. Begin with a simple call like `openai.chat.completions.create()`.
   - Store the API key in an environment variable or a config file (`utils/config.py`).

4. **Implement the Agent Loop**
   - Port the logic from `utils/agent/agent-loop.ts` to `agent/loop.py`.
   - Handle streaming, command approvals, and applying patches to files. Python's `subprocess` module can run shell commands. Wrap them in a sandbox (Docker or similar) for safety.

5. **Add a Rich/Textual Interface** (optional)
   - If you want a dynamic terminal UI similar to Ink, create components under `components/` using Rich or Textual widgets.
   - Keep the first version simple: print outputs and ask for confirmations via standard input.

6. **Testing**
   - Use `pytest` to mirror the existing test cases in `codex-cli/tests/`.
   - Run tests locally with `pytest` and keep them fast and focused.

7. **Packaging**
   - Expose a console script named `codex` in `pyproject.toml` so users can install with `pip install .` and run `codex` from the command line.
   - Document installation and usage in a new README.

## 5. Tips for Beginners

- Tackle the rewrite incrementally. Start with a minimal CLI that simply echoes the prompt, then gradually add features.
- Use type hints (`from __future__ import annotations` and `typing`) to keep the code readable and maintainable.
- Compare each new Python module with its TypeScript counterpart to ensure feature parity.
- Keep commits small and run tests often.

## 6. CLI Workflow Reference

If you want to see how a task flows through the existing TypeScript
implementation of the Codex CLI, check out
[`workflow.md`](workflow.md). It walks through what happens after you
launch `codex` with a prompt: from loading configuration, creating the
`AgentLoop`, streaming responses from the OpenAI API and executing shell
commands, all the way to writing the rollout log.

---

This guideline provides one possible path for porting the Codex CLI to Python. Feel free to adjust the structure and libraries to match your preferences or project requirements.

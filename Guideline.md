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
   pip install openai typer rich python-dotenv pytest
   ```
   - **Typer** builds friendly CLIs (based on Click).
   - **Rich** lets you print colored text and build simple text UIs. For more advanced terminal interfaces, consider **Textual**.
   - **python-dotenv** loads variables from a `.env` file so you can keep your OpenAI key out of the code.
   - **pytest** is used for writing and running tests.

## 3. Suggested Folder Structure

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

- `main.py` exposes a `typer.Typer` application. Define commands and options that mirror the original Node CLI (e.g., `--model`, `--quiet`, `--approval-mode`).
- `agent/loop.py` should handle the interaction with the OpenAI API using the `openai` Python SDK. Stream responses and execute shell commands inside a sandbox (e.g., Docker or a restricted subprocess).
- `utils/config.py` reads configuration files from `~/.codex/` (similar to the TypeScript version).
- `tests/` contains pytest tests that validate key behaviours.

## 4. Implementation Steps

1. **Bootstrap the package**
   - Create the folder structure above and add an empty `__init__.py` file to each package.
   - Use `pyproject.toml` (with Poetry or plain setuptools) to manage dependencies.

2. **CLI Invocation**
   - Build a Typer application in `cli/main.py` that parses options such as `--model` and `--quiet`.
   - Load variables from `.env` using `python-dotenv` and merge configuration files in `utils/config.py`.

3. **Create UI and AgentLoop**
   - Implement `AgentLoop` in `agent/loop.py` to handle interactions with the OpenAI API.
   - Render a basic chat interface from `ui/terminal.py` using Rich. Quiet mode can simply print the final answer.

4. **Send the First Prompt**
   - Convert the user prompt into input items (small dataclasses) and call `AgentLoop.run()` to start streaming with `openai.ChatCompletion.create`.

5. **Process Responses**
   - Display assistant messages as they stream in. When a tool call is approved, run the command via `subprocess` or a Docker sandbox and feed the output back as `function_call_output`.

6. **Loop and Finish**
   - Continue this cycle until the API signals completion or the user cancels. Save each conversation to `logs/` for later review.

7. **Testing**
   - Add pytest cases under `tests/` to keep behaviour stable. Run them with `pytest` regularly.

8. **Packaging**
   - Expose a console script named `codex` in `pyproject.toml` pointing to `codex_py.cli.main:app` so you can install the tool with `pip install .`.
   - Document installation and usage in a README.

## 5. Tips for Beginners

- Tackle the rewrite incrementally. Start with a minimal CLI that simply echoes the prompt, then gradually add features.
- Use type hints (`from __future__ import annotations` and `typing`) to keep the code readable and maintainable.
- Compare each new Python module with its TypeScript counterpart to ensure feature parity.
- Keep commits small and run tests often.

## 6. CLI Workflow Reference

If you want to see how a task flows through the existing TypeScript
implementation of the Codex CLI, check out
[`workflow.md`](codex-old/workflow.md). It walks through what happens after you
launch `codex` with a prompt: from loading configuration, creating the
`AgentLoop`, streaming responses from the OpenAI API and executing shell
commands, all the way to writing the rollout log.
For a Python oriented summary of the same process see
[`workflow_py.md`](workflow_py.md).

---

This guideline provides one possible path for porting the Codex CLI to Python. Feel free to adjust the structure and libraries to match your preferences or project requirements.

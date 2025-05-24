# Codex CLI Workflow

This document explains in detail how a task flows through the TypeScript implementation of the Codex CLI found in `codex-old`.

## 1. Launching the CLI
* Run `codex "<prompt>"` or execute one of the provided `run.sh` scripts.
* `src/cli.tsx` uses `meow` to parse command line options such as `--model` and `--quiet`.
* Environment variables from `.env` are loaded via `dotenv/config`. The OpenAI API key is taken from `OPENAI_API_KEY`.
* `loadConfig()` merges `~/.codex/instructions.md` with any project level `CODEX.md` to build the final configuration object.

## 2. Preparing the UI and Agent
* The CLI resolves the selected model, image paths and approval mode.
* An `App` React component is rendered with Ink to display the terminal chat interface.
* An `AgentLoop` instance is created with:
  - the chosen model and instructions,
  - callbacks for streaming output and showing loading spinners,
  - a function that asks the user to confirm commands depending on the approval policy.
* When `--quiet` is passed a simplified `runQuietMode()` routine uses the same `AgentLoop` without the Ink interface.

## 3. Sending the First Prompt
* The initial prompt and optional images are converted to `ResponseInputItem` objects via `createInputItem`.
* `AgentLoop.run()` is invoked with these items, creating a streaming request through `openai.responses.create()`.

## 4. Streaming and Function Calls
* The API returns `ResponseItem` events which may contain assistant messages or `function_call` objects.
* `terminal-chat.tsx` renders the messages. When a tool call arrives the user may be prompted to approve it.
* Approved shell commands run in `handleExecCommand` inside a sandbox; file edits are applied using `apply_patch`.
* Command results are packaged as `function_call_output` items and queued for the next API request so the model can continue.

## 5. Iteration Loop
* After sending tool output, `AgentLoop.run()` waits for the next batch of streamed responses.
* The cycle repeats: display messages, ask for confirmation, execute, and return output.
* Users can interrupt with `Ctrl-C`; `AgentLoop.cancel()` aborts any running commands and cleans up.

## 6. Completion and Log
* When the model signals completion or the user cancels, staged messages are flushed and the entire conversation is written to a rollout log under `$TMPDIR/oai-codex/`.
* Interactive sessions exit the Ink interface. Quiet mode simply prints the final assistant message.

See the sources in `codex-cli/src/cli.tsx`, `codex-cli/src/utils/agent/agent-loop.ts` and `codex-cli/src/components/chat/terminal-chat.tsx` for the full implementation details.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

O'Reilly live training course — **Context Engineering Hands-On** — teaching developers how to design, manage, and optimize the context that flows into LLMs and agentic systems. The repo holds slide decks and hands-on Python demos, all runnable via `uv run <script>.py` with an `ANTHROPIC_API_KEY` env var. `README.md` covers the same demos in more run-it-yourself detail (prerequisites, sample queries, per-demo troubleshooting); this file focuses on architecture. `README.md` doesn't cover the `make all`/`make clean` Jupyter-kernel workflow described below — that's dev-environment setup, not demo instructions.

## Commands

Each demo is a self-contained `uv` script (PEP 723 inline dependencies) or uses the repo-level venv — check each file's `# /// script` header for its own deps.

```bash
# Set API key (required by every demo)
export ANTHROPIC_API_KEY=sk-ant-...

# Run any standalone demo script
uv run demos/<path>/<script>.py

# Repo-level venv (Jupyter kernel, notebook work) — managed via Makefile, not uv run
make all            # create .venv, install deps, register Jupyter kernel
make add <package>  # add a package to requirements.in and resync
make freeze          # snapshot current venv into requirements/requirements.txt
make clean            # remove .venv and unregister the Jupyter kernel
```

There is no test suite and no CI enforcement — it's course material, not a shipped library. Ruff is used ad hoc to keep the demos clean (`uvx ruff check demos`); deliberate exceptions (e.g. an intentionally broad `except Exception` in a REPL loop) are marked with `# noqa: <RULE> — <reason>` rather than fixed.

## Structure

```text
presentation-slides/   # Slide decks (.html — remark.js live + handout)
assets/                # Reference PDFs
demos/
  agentic-retrieval/                     # Hand-rolled agent loop with TF-IDF retrieval
  chat-with-artifacts/                   # FastAPI app with structured-output artifacts
  live-demo-chat-agent-ctx-eng-overview/ # Standalone scripts: file-tool chat agent, quiz app, structured-output primer
  full_agent_app.py                      # Agentic-RAG CLI built on claude-agent-sdk
```

## Architecture

Every demo teaches a different context-engineering lever by making it visible in ~100-300 lines of code, deliberately avoiding framework magic. When editing any demo, preserve this transparency — don't hide the `messages` array or tool-result plumbing behind an abstraction.

### `demos/agentic-retrieval/` — the core teaching module

- `agent.py` — hand-rolled agent loop. `Agent.messages` **is** the context window; `run_turn()` shows exactly how user input, assistant tool-use blocks, and tool results accumulate in that list across up to `MAX_TOOL_ROUNDS` iterations. Inline `# ─── TEACHING MOMENT ───` comments mark the pedagogically important lines — preserve/extend that comment style if you touch this file.
- `retrieval.py` — TF-IDF search over `knowledge_base/*.md` (the "selecting context" lever).
- `tools.py` — Anthropic tool schemas + dispatch (`search_documents`, `get_document`, `list_documents`).
- `display.py` — ANSI terminal rendering for chat, tool calls, and stats.
- `app.py` — TUI entry point with slash commands (`/context`, `/stats`, `/clear`, `/docs`, `/help`, `/quit`).

### `demos/chat-with-artifacts/`

- `app.py` — FastAPI backend. System prompt is built from 3 layers with distinct token costs: `BASE_PERSONA` (fixed), artifact descriptions (static per session), and dynamic session state (grows as artifacts are created) — see the `TEACHING MOMENT` block near `MODEL`/`BASE_PERSONA`.
- `schemas.py` — `ARTIFACT_REGISTRY` defines all artifact types (description, `when_to_use` routing hint, JSON schema). The registry both drives the single `create_artifact` tool's structured output and feeds dynamic context back into the system prompt each turn.
- `static/index.html` — single-file frontend.
- `structured_outputs_demo.py` — standalone, no-FastAPI primer on forcing one prompt through 3 different tool schemas.

### `demos/live-demo-chat-agent-ctx-eng-overview/`

Independent single-file scripts, not a package:

- `chat.py` — minimal agent loop with `web_search`, `read_file`, `create_file` tools.
- `quiz_app.py` — FastAPI + `client.messages.parse(..., output_format=Quiz)` structured-output quiz generator with inline HTML frontend.
- `structured_output_example.py` — smallest possible `messages.parse` structured-output example.

### `demos/full_agent_app.py`

Agentic-RAG CLI using `claude_agent_sdk` (`ClaudeSDKClient`, `@tool`, `create_sdk_mcp_server`) instead of raw `anthropic` calls. Points at `demos/agentic-retrieval/knowledge_base/` by default; accepts an alternate folder as `argv[1]`. Defines `list_docs`/`read_doc`/`search_docs` as SDK tools — useful as the "framework does the loop for you" counterpart to `agentic-retrieval/agent.py`'s hand-rolled loop.

## Conventions across demos

- Model constant is `MODEL = "claude-sonnet-4-6"` (or hardcoded inline) — keep new demos consistent unless intentionally demonstrating a different model.
- API key loading: FastAPI-based demos call `load_dotenv()` before importing `anthropic`; plain scripts read `ANTHROPIC_API_KEY` from the environment directly.
- PEP 723 inline script headers (`# /// script` ... `# ///`) are authoritative for *running* each standalone demo via `uv run` — check/update these when adding imports to a demo. `requirements/requirements.in` (backing the repo-level `.venv` via `Makefile`) separately mirrors the union of every demo's third-party deps, purely so the Jupyter kernel and editor tooling (type checking, import resolution) can see across all demos at once — keep the two in sync by hand when either changes, but don't conflate them: the `.venv` is never what actually runs a demo.

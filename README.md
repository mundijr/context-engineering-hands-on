> **TL;DR** — 7 runnable demos across 3 directories, all runnable with [uv](https://docs.astral.sh/uv/) and an Anthropic API key.
> `export ANTHROPIC_API_KEY=… && uv run demos/<path>/<script>.py`

# Context Engineering Hands-On

**O'Reilly Live Training** — Teaching developers how to design, manage, and optimize the context that flows into LLMs and agentic systems.

Live demos and hands-on code — all runnable with [uv](https://docs.astral.sh/uv/) and an Anthropic API key.

---

## Demos at a Glance

| Demo | Directory | What it shows |
|------|-----------|----------------|
| Agentic document retrieval | `demos/agentic-retrieval/` | Hand-rolled agent loop, TF-IDF retrieval, explicit context management |
| Chat with artifacts | `demos/chat-with-artifacts/` | FastAPI app, 3-layer system prompt, structured tool output |
| Minimal file-tool chat agent | `demos/live-demo-chat-agent-ctx-eng-overview/chat.py` | Smallest possible agent loop with `web_search`, `read_file`, `create_file` tools |
| Quiz app | `demos/live-demo-chat-agent-ctx-eng-overview/quiz_app.py` | FastAPI + `messages.parse` structured-output quiz generator |
| Structured output primer | `demos/live-demo-chat-agent-ctx-eng-overview/structured_output_example.py` | Smallest possible `messages.parse` example |
| Structured outputs primer (bonus) | `demos/chat-with-artifacts/structured_outputs_demo.py` | Same prompt forced through 3 different tool schemas, no FastAPI scaffolding |
| Agentic-RAG CLI | `demos/full_agent_app.py` | Same retrieval idea as `agentic-retrieval/`, built on `claude-agent-sdk` instead of raw API calls |

---

## Prerequisites

- **Python 3.12+**
- **[uv](https://docs.astral.sh/uv/)** — the package manager used by every demo
- **Anthropic API key** — get one at [console.anthropic.com](https://console.anthropic.com)

### Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Set your API key

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

Add it to your shell profile (`~/.zshrc` or `~/.bashrc`) to persist it across sessions. Alternatively, create a `.env` file in any demo directory:

```bash
echo "ANTHROPIC_API_KEY=sk-ant-..." > demos/<demo-name>/.env
```

### Clone the repo

```bash
git clone <REPO_URL>
cd context-engineering-hands-on
```

No global `pip install` or virtual environment needed — every demo uses `uv run` to manage its own dependencies inline.

---

## Demos

### Agentic Document Retrieval

**Directory:** `demos/agentic-retrieval/`

A hand-rolled agent loop with explicit context management and TF-IDF document retrieval. Every context engineering decision is visible — no framework magic. See `demos/agentic-retrieval/README.md` for the full architecture diagram, suggested query sequence, and slash-command reference.

**Run:**

```bash
cd demos/agentic-retrieval
uv run app.py
```

**Try:** `What documents do you have?`, then `/context` to inspect the raw messages array. The full query sequence and slash-command reference live in `demos/agentic-retrieval/README.md`.

---

### Chat with Artifacts

**Directory:** `demos/chat-with-artifacts/`

A FastAPI backend + single-file frontend demonstrating a 3-layer system prompt, structured tool output, and a growing artifact registry injected into context.

**Run:**

```bash
cd demos/chat-with-artifacts
uv run app.py
```

Then open **http://127.0.0.1:8001** in your browser.

**What to try:**
- Ask about any topic to see the artifact system in action
- Watch the token counter grow in the stats bar as artifacts are created
- Ask to be quizzed to see context from earlier turns referenced in new outputs

**What's demonstrated:**
- 3-layer system prompt (persona + artifact schemas + dynamic session state)
- Structured output via a single `create_artifact` tool
- Dynamic context injection — the artifact registry grows and re-enters the system prompt
- Conversation history as accumulating context

**Bonus — minimal structured-outputs primer:** `structured_outputs_demo.py` in the same directory is a standalone ~30-second read showing the same prompt forced through 3 different tool schemas (no FastAPI scaffolding). Run with `uv run structured_outputs_demo.py`.

---

### Live Demo: Chat Agent Overview

**Directory:** `demos/live-demo-chat-agent-ctx-eng-overview/`

Three independent single-file scripts, not a package. `cd demos/live-demo-chat-agent-ctx-eng-overview` first, then:

- **`chat.py`** — minimal agent loop with `web_search`, `read_file`, and `create_file` tools. Executable directly (`./chat.py`) or via `uv run chat.py`.
- **`quiz_app.py`** — a tiny chat-driven quiz generator. Run `uv run quiz_app.py`, then open **http://127.0.0.1:8501**. Builds on `structured_output_example.py` by rendering questions in the browser and auto-grading answers.
- **`structured_output_example.py`** — the smallest possible `messages.parse` structured-output example. Run `uv run structured_output_example.py`.

---

### Agentic-RAG CLI (Agent SDK)

**File:** `demos/full_agent_app.py`

A chat app where Claude has three custom tools for working with a folder of `.md` files: list them, read one, or search across them. Built on `claude-agent-sdk` instead of raw `anthropic` API calls — a useful contrast to the hand-rolled loop in `agentic-retrieval/agent.py`.

**Run:**

```bash
uv run demos/full_agent_app.py
# or point it at a different folder of markdown files:
uv run demos/full_agent_app.py path/to/folder
```

Defaults to `demos/agentic-retrieval/knowledge_base/` if no folder is given.

---

## Repo Structure

```
context-engineering-hands-on/
├── presentation-slides/                       # Slide decks (.html — remark.js live + handout)
├── assets/                                    # Reference PDFs (attention paper, cheatsheets)
└── demos/
    ├── agentic-retrieval/                     # Hand-rolled agent loop + TF-IDF retrieval
    │   ├── app.py                             # Entry point (TUI + slash commands)
    │   ├── agent.py                           # Agent loop — core teaching file
    │   ├── retrieval.py                       # TF-IDF document search
    │   ├── tools.py                           # Tool schemas + dispatch
    │   ├── display.py                         # ANSI terminal output
    │   └── knowledge_base/                    # 6 markdown docs on course topics
    ├── chat-with-artifacts/                   # FastAPI app with structured artifact output
    │   ├── app.py                             # FastAPI backend
    │   ├── schemas.py                         # Artifact type schemas
    │   ├── static/index.html                  # Single-file frontend
    │   └── structured_outputs_demo.py         # Bonus: minimal structured-outputs primer
    ├── live-demo-chat-agent-ctx-eng-overview/ # Standalone single-file scripts
    │   ├── chat.py                            # Minimal agent loop (web_search, read_file, create_file)
    │   ├── quiz_app.py                        # FastAPI chat-driven quiz generator
    │   └── structured_output_example.py       # Smallest structured-output example
    └── full_agent_app.py                      # Agentic-RAG CLI built on claude-agent-sdk
```

---

## Troubleshooting

**`ANTHROPIC_API_KEY` not found**

```bash
# Check if it's set
echo $ANTHROPIC_API_KEY

# Set it for the current session
export ANTHROPIC_API_KEY=sk-ant-...
```

**`uv: command not found`**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# Then restart your terminal
```

**Port 8001 already in use (chat-with-artifacts)**

```bash
# Find and kill the process using port 8001
lsof -ti:8001 | xargs kill -9
# Then re-run
uv run app.py
```

---

*Course slides and reference materials are in `presentation-slides/` and `assets/`.*

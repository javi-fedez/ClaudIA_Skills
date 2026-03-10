# CLAUDE.md — ClaudIA Agent

This file provides guidance for AI assistants (Claude Code and others) working on the **ClaudIA** codebase. Read this before making any changes.

---

## Project Overview

**ClaudIA** is an autonomous, multipurpose AI agent built on top of Anthropic's Claude models. It is designed to operate independently, decompose complex tasks, use tools, and deliver results — similar in spirit to projects like OpenManus, OpenClaw, or AutoGPT, but tailored around Claude's strengths.

**Goal:** A self-directed Claude agent that can handle research, coding, file manipulation, web browsing, API calls, and multi-step planning without constant human intervention.

---

## Repository State

| Item | Status |
|------|--------|
| Source code | Not yet created |
| Dependencies | Not yet defined |
| Tests | Not yet configured |
| CI/CD | Not yet configured |
| Docs | README.md only |

This is a greenfield project. The conventions below define how it **should** be built.

---

## Planned Architecture

```
ClaudIA/
├── CLAUDE.md              # This file — AI assistant guide
├── README.md              # Human-facing project overview
├── pyproject.toml         # Python project config & dependencies
├── .env.example           # Template for required environment variables
├── .gitignore
│
├── claudia/               # Main Python package
│   ├── __init__.py
│   ├── agent.py           # Core agent loop
│   ├── config.py          # Configuration loading
│   ├── memory/            # Memory subsystems
│   │   ├── short_term.py  # In-context memory
│   │   └── long_term.py   # Persistent vector/DB memory
│   ├── tools/             # Tool registry and implementations
│   │   ├── registry.py    # Tool loader and dispatcher
│   │   ├── web_search.py
│   │   ├── code_exec.py
│   │   ├── file_ops.py
│   │   └── http_client.py
│   ├── planner.py         # Task decomposition & planning
│   ├── safety.py          # Guardrails and content policy
│   └── cli.py             # Command-line entrypoint
│
├── tests/
│   ├── conftest.py
│   ├── test_agent.py
│   ├── test_tools.py
│   └── test_memory.py
│
└── docs/
    └── architecture.md
```

---

## Technology Decisions

| Concern | Choice | Rationale |
|---------|--------|-----------|
| Language | Python 3.11+ | Best LLM/AI ecosystem |
| LLM SDK | `anthropic` (official) | Native Claude support, tool use, streaming |
| Preferred model | `claude-sonnet-4-6` | Best capability/cost balance |
| Fallback model | `claude-haiku-4-5-20251001` | Fast/cheap for subtasks |
| Package manager | `uv` | Fast, modern Python tooling |
| Testing | `pytest` | Standard, well-supported |
| Linting | `ruff` | Fast linter + formatter |
| Type checking | `mypy` | Static type safety |
| Env management | `python-dotenv` | Standard `.env` loading |
| Memory (vector) | `chromadb` | Local-first, easy to use |

---

## Development Workflow

### Setup

```bash
# Install uv if not present
curl -Ls https://astral.sh/uv/install.sh | sh

# Create virtual env and install deps
uv venv
source .venv/bin/activate
uv pip install -e ".[dev]"

# Copy environment template
cp .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY
```

### Running the agent

```bash
python -m claudia.cli "Research and summarize the latest Claude 4 benchmarks"
```

### Running tests

```bash
pytest
pytest --cov=claudia --cov-report=term-missing   # with coverage
```

### Linting & formatting

```bash
ruff check .
ruff format .
mypy claudia/
```

---

## Key Conventions

### Code Style

- Use **Python type hints** everywhere — functions, class attributes, return types.
- Use `dataclass` or `pydantic.BaseModel` for structured data; never raw dicts for public APIs.
- Keep functions small and single-purpose. Aim for < 40 lines per function.
- All public functions and classes must have docstrings.
- Prefer `pathlib.Path` over `os.path`.
- Use `logging` (not `print`) for all diagnostic output.

### Naming

| Item | Convention | Example |
|------|------------|---------|
| Files/modules | `snake_case` | `web_search.py` |
| Classes | `PascalCase` | `ToolRegistry` |
| Functions/vars | `snake_case` | `run_agent_loop()` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_ITERATIONS = 20` |
| Private members | `_leading_underscore` | `_build_system_prompt()` |

### Error Handling

- Never swallow exceptions silently.
- Use specific exception types, not bare `except Exception`.
- Log errors with full context before re-raising or recovering.

### Git Workflow

- Branch naming: `claude/<short-description>-<session-id>` for AI-driven work; `feature/<name>` for human-driven work.
- Commit messages: imperative mood, ≤72 chars subject line.
  - Good: `Add web search tool with retry logic`
  - Bad: `added stuff` / `WIP`
- Never commit `.env`, secrets, or API keys.
- Always run `ruff` and `pytest` before pushing.

### Environment Variables

All secrets and config go in `.env` (never committed). Document every variable in `.env.example`.

Required variables:
```
ANTHROPIC_API_KEY=          # Required — Anthropic API key
CLAUDIA_MAX_ITERATIONS=20   # Max agent loop iterations
CLAUDIA_MODEL=claude-sonnet-4-6
CLAUDIA_LOG_LEVEL=INFO
```

---

## Agent Design Principles

1. **Tool-first**: The agent should never "hallucinate" information it could look up. Prefer calling a tool.
2. **Transparent reasoning**: Log every decision step. Users must be able to audit what the agent did.
3. **Fail safely**: If uncertain or if an action is irreversible (delete, send, publish), pause and confirm.
4. **Idempotent tools**: Tools should be safe to retry. Use checksums/dedup where possible.
5. **Budget awareness**: Track token usage per run. Abort gracefully if limits are approached.
6. **Minimal footprint**: Don't write files or make network calls unless the task requires it.

---

## Anthropic SDK Usage

Always use the official `anthropic` SDK. Refer to the `claude-api` skill for detailed patterns.

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8192,
    tools=[...],          # tool definitions go here
    messages=[{"role": "user", "content": "..."}],
)
```

- Use **tool use** (function calling) for all structured agent actions.
- Use **streaming** (`client.messages.stream(...)`) for long-running tasks to provide live feedback.
- Prefer `claude-sonnet-4-6` for orchestration; `claude-haiku-4-5-20251001` for cheap subtasks.

---

## Testing Guidelines

- Every tool must have unit tests that mock the external call.
- Agent loop tests should use recorded fixtures, not live API calls.
- Use `pytest-vcr` or manual mocking for HTTP dependencies.
- Aim for > 80% coverage on `claudia/` core modules.

---

## What AI Assistants Should NOT Do

- Do not add unrequested features or abstractions.
- Do not push to `main` or `master` — always use feature branches.
- Do not commit secrets or API keys.
- Do not make destructive git operations without explicit user approval.
- Do not use `print()` for logging in production code — use the `logging` module.
- Do not add backwards-compatibility shims for code that doesn't exist yet.

---

## Built-in Skills Documentation

ClaudIA leverages Claude Code's **native tools** without requiring third-party APIs. Each skill is documented in `docs/skills/`:

| Skill | File | Descripción |
|-------|------|-------------|
| Web Research | `docs/skills/web-research.md` | Búsqueda web + extracción de contenido de URLs |
| File Operations | `docs/skills/file-ops.md` | Leer, escribir, editar, buscar archivos |
| Code Execution | `docs/skills/code-execution.md` | Scripts Python, automatizaciones shell |
| Content Creation | `docs/skills/content-creation.md` | Propuestas, emails, posts, materiales de curso |
| Data Analysis | `docs/skills/data-analysis.md` | Análisis de CSV/JSON, informes ejecutivos |
| Agent Orchestration | `docs/skills/agent-orchestration.md` | Sub-agentes paralelos para tareas complejas |
| Git Operations | `docs/skills/git-operations.md` | Versionado documental, historial, recuperación |

## CEO Task Menu

For autonomous business operation tasks targeted at a small training/education company CEO, see:

**`docs/ceo-menu.md`** — 20 ready-to-use task prompts covering:
- Commercial & Sales (propuestas, pipeline, análisis)
- Marketing & Communication (calendario editorial, newsletters, LinkedIn)
- Operations & Management (informes, OKRs, onboarding)
- Product & Training (diseño de cursos, actualización de materiales)
- Finance & Administration (facturación, rentabilidad, gestoría)
- Strategy & Growth (captación, DAFO, planificación)

---

## References

- [Anthropic API Docs](https://docs.anthropic.com)
- [Claude Tool Use Guide](https://docs.anthropic.com/en/docs/tool-use)
- [Anthropic Python SDK](https://github.com/anthropics/anthropic-sdk-python)

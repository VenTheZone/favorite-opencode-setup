# My OpenCode Setup

This is my personal OpenCode config. I've tweaked it over time to balance speed, cost, and quality. It uses a mix of free models (Kilo, Cline) and paid ones (OpenAI, Modal) depending on what I need done.

## What's Inside

- **opencode.json** - Core agent definitions and provider configs
- **oh-my-opencode-slim.json** - Dynamic presets with fallback chains when models fail

## The Agents

### Subagents (defined in opencode.json)

These are the specialist agents I call for specific jobs:

| Agent | Model | What It Does |
|-------|-------|--------------|
| planner | GLM-5 Free (Cline) | Breaks down complex features into actionable steps |
| architect | GLM-5-FP8 (Modal) | Big-picture system design and technical decisions |
| brainstormer | MiniMax M2.5 Free (Kilo) | Creative ideation and exploring alternatives |
| code-reviewer | GPT-5.4 | General code quality and maintainability checks |
| security-reviewer | GPT-5.4 | Catches security issues in auth, APIs, sensitive data |
| tdd-guide | GPT-5.3-codex | Enforces test-first development, coverage checks |
| build-error-resolver | GPT-5.3-codex-spark | Fast fixes for build/type errors |
| e2e-runner | GPT-5.3-codex | Playwright test generation and maintenance |
| doc-updater | GPT-5.4 | Keeps documentation and codemaps current |
| refactor-cleaner | MiniMax M2.5 Free (Kilo) | Removes dead code and consolidates duplicates |
| go-reviewer | GPT-5.4 | Go-specific idioms, concurrency, performance |
| go-build-resolver | GPT-5.3-codex-spark | Quick Go compilation fixes |
| database-reviewer | GPT-5.4 | PostgreSQL query optimization and schema review |
| rust-reviewer | GPT-5.4 | Rust ownership, lifetimes, safety patterns |
| rust-clippy-fmt-check-tester | GPT-5.3-codex-spark | Rust tooling error resolution |

### Dynamic Presets (oh-my-opencode-slim.json)

These are the orchestrator-level agents that manage the workflow:

| Agent | Primary Model | Key Skills |
|-------|---------------|------------|
| oracle | GPT-5.4 (high) | context7-base-code-review, visual-explainer |
| orchestrator | GLM-5 Free (Cline) | dispatching-parallel-agents, cartography, writing-plans, git-worktrees, verification |
| fixer | GPT-5.3-codex (low) | systematic-debugging, context7-driven-dev |
| designer | GPT-5.4 (medium) | visual-explainer, agent-browser |
| librarian | MiniMax M2.5 Free (Kilo, low) | context7-base-code-review, cartography, visual-explainer |
| explorer | MiniMax M2.5 Free (Kilo, low) | cartography, context7-base-code-review, systematic-debugging |

## Providers I'm Using

- **Kilo** - OpenCode's gateway to free models (MiniMax M2.5 Free)
- **Cline** - Free tier with GLM-5 Free, MiniMax, and KAT Coder Pro
- **Modal** - GLM-5-FP8, a 744B parameter model
- **OpenAI** - GPT-5.4, GPT-5.3-codex, GPT-5.3-codex-spark (business plan)
- **Kimi** - Moonshot's kimi-k2.5 for orchestration fallback

## Fallback Chains

When a model times out (15s) or fails, it tries the next one in line:

| Agent | Fallback Order |
|-------|----------------|
| oracle | GPT-5.4 → GLM-5 Free → kimi-k2.5 → GPT-5.3-codex → MiniMax M2.5 Free → GPT-5.3-codex-spark |
| orchestrator | GLM-5 Free → kimi-k2.5 → GPT-5.4 → MiniMax M2.5 Free → GPT-5.3-codex → GPT-5.3-codex-spark |
| fixer | GPT-5.3-codex → GPT-5.3-codex-spark → kimi-k2.5 → GPT-5.4 → GLM-5 Free → MiniMax M2.5 Free |
| designer | GPT-5.4 → MiniMax M2.5 Free → GLM-5 Free → kimi-k2.5 → GPT-5.3-codex → GPT-5.3-codex-spark |
| librarian | MiniMax M2.5 Free → GPT-5.4 → GLM-5 Free → kimi-k2.5 → GPT-5.3-codex → GPT-5.3-codex-spark |
| explorer | MiniMax M2.5 Free (Kilo) → GPT-5.3-codex → GPT-5.3-codex-spark → kimi-k2.5 → MiniMax M2.5 Free → GLM-5 Free → GPT-5.4 |
| refactor-cleaner | MiniMax M2.5 Free (Kilo) → MiniMax M2.5 Free (Cline) |

Explorer and refactor-cleaner start with free models since they run frequently for file searches and cleanup tasks.

## Setup

Drop these into your OpenCode config:

```bash
cp opencode.json ~/.config/opencode/
cp oh-my-opencode-slim.json ~/.config/opencode/
```

Then add your API keys to `~/.local/share/opencode/auth.json`:

```json
{
  "kilo": { "type": "api", "key": "your-kilo-key" },
  "cline": { "type": "api", "key": "your-cline-key" },
  "openai": { "type": "api", "key": "your-openai-key" },
  "modal": { "type": "api", "key": "your-modal-key" },
  "kimi": { "type": "api", "key": "your-kimi-key" },
  "openrouter": { "type": "api", "key": "your-openrouter-key" }
}
```

### Free API keys

- **Kilo**: Through OpenCode's gateway, no separate signup
- **Cline**: https://app.cline.bot

OpenAI, Modal, and Kimi require paid subscriptions.

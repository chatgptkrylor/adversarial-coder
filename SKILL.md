---
name: adversarial-coder
description: GAN-inspired three-agent harness for reliable AI coding. Uses Codex as Generator (builds) and Kimi/Gemini as Planner/Evaluator (tries to break). Separates planning, building, and evaluation into distinct agents with distinct contexts. Use for complex multi-sprint projects where quality matters more than speed. NOT for quick fixes or simple tasks.
metadata:
  {
    "openclaw":
      {
        "emoji": "⚔️",
        "requires": { "bins": ["codex", "kimi", "gemini"] },
      },
  }
---

# Adversarial Coder

A GAN-inspired harness that separates **planning**, **building**, and **evaluation** into distinct agents. The evaluator's job is to **break** what the generator builds — creating adversarial tension that drives quality far beyond what a single agent can achieve.

## Why This Works

Most AI coding agents fail on complex tasks because a single agent that plans, builds, and evaluates its own work will reliably praise its own mediocre output. This is called **self-evaluation bias**.

This harness implements the fix: three agents, each with a focused job.

| Agent | Role | Model |
|-------|------|-------|
| **Planner** | Expands prompt into spec with sprints | Gemini (fast, cheap) |
| **Generator** | Builds features, commits to git | Codex (best coder) |
| **Evaluator** | Tries to break the implementation | Gemini (adversarial) |

The evaluator doesn't just review code — it's an adversary. It runs the application, probes for failures, tests edge cases, and scores ruthlessly. If any criterion fails (score < 7/10), the sprint goes back to the generator with detailed feedback.

## Quick Start

```bash
# One-shot task
cd /home/krylor/.openclaw/workspace
adversarial-coder "Build a CLI tool that searches PubChem for compound information"

# From a prompt file
adversarial-coder --file prompt.md
```

## Configuration

Defaults can be overridden via environment variables or flags:

| Setting | Default | Description |
|---------|---------|-------------|
| `maxSprints` | 10 | Maximum number of sprints |
| `maxRetriesPerSprint` | 3 | Max evaluation retries before failing |
| `passThreshold` | 7 | Minimum score (out of 10) for each criterion |
| `TIMEOUT` | 300 | Timeout per agent call in seconds |
| `CODEX_MODEL` | gpt-5.4 | Model for Generator |
| `GEMINI_MODEL` | gemini-2.5-flash | Model for Planner/Evaluator |

## The Flow

```
User Prompt (1-4 sentences)
         |
         v
   +-----------+
   |  PLANNER  |  --> writes spec.md (features, sprints, design language)
   +-----------+
         |
         v  (for each sprint)
   +---------------------+
   | CONTRACT NEGOTIATION |  Generator proposes criteria,
   | Generator <-> Eval   |  Evaluator tightens the screws,
   +---------------------+  both lock in "done"
         |
         v
   +-----------+     fail + feedback     +------------+
   | GENERATOR | <---------------------- | EVALUATOR  |
   | (build)   | ----------------------> | (attack)   |
   +-----------+     implementation      +------------+
         |                                      |
         v              pass                    |
    Next Sprint <-------------------------------+
```

## Sprint Contracts (The Innovation)

Before coding, Generator and Evaluator negotiate a **JSON contract**:

```json
{
  "sprintNumber": 1,
  "features": ["REST API endpoints", "SQLite database layer"],
  "criteria": [
    {
      "name": "api_get_conversations",
      "description": "GET /api/conversations returns 200 with array of conversation objects",
      "threshold": 7
    },
    {
      "name": "error_handling",
      "description": "API returns proper error codes (400, 404, 500) with JSON error messages",
      "threshold": 7
    }
  ]
}
```

Evaluator uses contract negotiation to **set traps** — adding edge cases, tightening thresholds, demanding specifics.

## Files Created

```
workspace/
├── spec.md                    # Product specification from Planner
├── contracts/
│   └── sprint-{n}.json        # Negotiated sprint contracts
├── feedback/
│   └── sprint-{n}-round-{m}.json  # Evaluator feedback per attempt
├── progress.json              # Current harness state
└── app/                       # Built application
```

## Requirements

- **Codex CLI**: `npm install -g @openai/codex`
- **Kimi CLI** (preferred): `npm install -g @anthropic-ai/kimi-cli` or `brew install kimi-cli`
- **Gemini CLI** (fallback): `brew install gemini-cli` or `npm install -g @anthropic-ai/gemini-cli`
- **Git**: For commit tracking

The harness auto-detects Kimi (preferred for agentic workflows) or falls back to Gemini.

## The Scoring System

- **9-10**: Exceptional. Works perfectly, handles edge cases.
- **7-8**: Good. Core functionality works. (PASS threshold)
- **5-6**: Partial. Some functionality, significant gaps.
- **3-4**: Poor. Fundamental issues.
- **1-2**: Failed. Not implemented or broken.

**All criteria must pass (≥7) or sprint fails.**

## Usage Examples

### Simple CLI Tool

```bash
adversarial-coder "Build a chemfind CLI that searches PubChem by name or CAS number"
```

### Complex Full-Stack App

```bash
adversarial-coder --file prompt.md
```

Where `prompt.md` contains:
```markdown
Build a full-stack RAG chat application with:
- React + Vite + TypeScript frontend
- Python + FastAPI backend
- SQLite database with repository pattern
- Dark theme with electric blue accents
- Streaming responses via SSE
```

### Resume After Failure

If a sprint fails after max retries, the harness stops. You can:
1. Read `feedback/sprint-{n}-round-{m}.json` to see what failed
2. Fix issues manually, or
3. Re-run with adjusted criteria

## Tips

1. **Start with clear prompts** — The better the initial description, the better the spec
2. **Check `spec.md` after planning** — Make sure the Planner understood your intent
3. **Watch the contracts** — Each sprint contract defines exactly what "done" means
4. **Read feedback files** — Evaluator gives specific file paths, line numbers, error messages
5. **Use for complex projects** — Don't use for quick one-liner fixes

## Architecture Notes

This is inspired by **Generative Adversarial Networks (GANs)**:
- GANs: Generator vs. Discriminator
- This: Generator vs. Evaluator (with Planner added)

The key insight: **separate generation from evaluation, then pit them against each other.**

From Anthropic's engineering article:
> "Every component in a harness encodes an assumption about what the model can't do on its own."

As models improve, harnesses simplify. But the **pattern** of separating roles is durable.

---

_This skill implements the adversarial-dev architecture from https://github.com/chatgptkrylor/adversarial-dev_